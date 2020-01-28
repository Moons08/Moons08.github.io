---
title: \[Airflow] Subdag 활용하기 
date: 2019-08-10
tags: airflow 
category: programming
sidebar:
    nav: "airflow"
---

재사용할 여지가 많은 task들을 묶어 subdag로 만들어 보겠습니다. 이렇게 하면 지저분한 과정들을 묶어서 한눈에 프로세스를 파악하기도 편하고, 관리도 쉬워집니다.

## Subdag

에어플로우에서는 스케쥴링할 작업을 DAG단위로 구분합니다. 프로세스를 만들다보면 반복되면서 지저분해지는 task 들을 생성할 때가 있는데, 이런 경우에 subdag를 이용하기 좋습니다. 아래 예시와 같이 사용할 수 있습니다.

![img](/assets/img/post/airflow/maindag.png)  
*subdag를 사용하는 dag의 graph view*

![img](/assets/img/post/airflow/toSubdag.png)  
*subdag operator를 클릭하면 subdag의 detail을 볼 수 있는 탭이 나옵니다.*

![img](/assets/img/post/airflow/subdag.png)  
*위와 같습니다.*

### Example

먼저 subdag를 파이썬 함수로 define 해야 합니다. 그리고 메인 dag에서 import 합시다.

```python
# subdag.py

from airflow.models import DAG
from airflow.operators.dummy_operator import DummyOperator
from airflow.operators.bash_operator import BashOperator
def random_subdag(parent_dag_name, child_dag_name, args):
    dag_subdag = DAG(
        dag_id='%s.%s' % (parent_dag_name, child_dag_name), # subdag의 id는 이와같은 컨벤션으로 쓴답니다.
        default_args=args,
        schedule_interval=None, # 값을 넣어주어야 합니다.
    )
    union =DummyOperator(
        task_id='%s-union' % (child_dag_name),
        default_args=args,
        dag=dag_subdag
        )

    for i in range(2):
        globals()['process_a'] = BashOperator(
            task_id='%s-task_A-%s' % (child_dag_name, i + 1),
            default_args=args,
            bash_command='echo "does it work?"',
            dag=dag_subdag,
        )
        globals()['process_b'] = BashOperator(
            task_id='%s-task_B-%s' % (child_dag_name, i + 1),
            default_args=args,
            bash_command='date',
            dag=dag_subdag,
        )

        process_a >> process_b >> union

    return dag_subdag
```

이 subdag는 병렬로 `process_a` 작업 후  `process_b`을 하고, 모든 `b` 작업이 완료되면 `union` 작업을 합니다.  

`default_args`들은 maindag의 것을 파라미터로 받아서 없어도 되나 했는데 해당 값을 안주면 작동을 안하더군요.

>공식 문서는 심지어 '@once' 나 'None'으로 설정하면 작업도 안하고 성공처리 된다고 합니다만, 진짜 그런가? 하고 해봤더니 작동해서 혼란 중입니다. 아무튼 값을 넣기는 하는걸로.
  
이렇게 만든 함수를 maindag에서 사용합니다.

```python
# maindag.py

from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from datetime import datetime, timedelta
from airflow.operators.subdag_operator import SubDagOperator
from mk_subdag import random_subdag

default_args = {
    "owner": "mk",
    "depends_on_past": False,
    "start_date": datetime(2019, 7, 1),
    "retries": 1,
}
dag = DAG("mk_subdag_tutorial"
        ,default_args=default_args
        ,schedule_interval="@once")

# t1, t2 and t3 are examples of tasks created by instantiating operators
t1 = BashOperator(task_id="print_date", bash_command="date", dag=dag)
t2 = BashOperator(task_id="sleep", bash_command="sleep 5", retries=3, dag=dag)

templated_command = """
{% raw %}
    {% for i in range(5) %}
        echo "{{ ds }}"
        echo "{{ macros.ds_add(ds, 7)}}"
        echo "{{ params.my_param }}"
    {% endfor %}
{% endraw %}
"""

t3 = BashOperator(
    task_id="templated",
    bash_command=templated_command,
    params={"my_param": "Parameter I passed in"},
    dag=dag,
)
# 여기까지는 도커로 설치하기에서 딸려오는 예제 tuto.py에 있는 부분을 가져왔습니다.

t4 = SubDagOperator( # 여기서 아까 만든 애를 불러옵니다
    task_id='sub_dag',
    subdag=random_subdag(dag.dag_id, 'sub_dag', default_args),
    default_args=default_args,
    dag=dag,
)

t2.set_upstream(t1)
t3.set_upstream(t1)

t3 >> t4
```

해야할 설정은 끝입니다. 이제 webserver로 들어가서 maindag와 subdag를 `on`으로 바꿔줍시다.  
공식문서에서는 subdag도 꼭 `on` 해주라고 합니다.

>안하고 작동시켜도 되던데... 확실하게 하기 위해서, 켜줍시다. `schedule_interval`부터 뭔가 요상하네요.

---

요약하자면,

1. Subdag 함수를 define한다.
1. subdag operator에서 define한 함수를 불러온다.
  
이제 반복되는(혹은 지저분한) 코드를 subdag에 숨길 수 있습니다.
