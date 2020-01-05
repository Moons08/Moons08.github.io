---
title: \[Airflow] Branch로 상황에 맞는 작업 수행하기
date: 2019-03-17
tags: airflow
category: airflow
sidebar:
    nav: "airflow"
---

선행 작업의 결과에 따라 이어나갈 작업이 달라야 할 때는 Branch로 분기를 나누어 줄 수 있습니다.

#### 예상 상황

- 데이터 입수 후 검증
  - 데이터에 `이상 징후`가 포착될 경우, 추가 전처리 작업 실행
  - 아닐 경우 해당 전처리 작업 Skip

- 모델 예측 후 적용
  - 모델 예측 결과가 기준치 이하일 경우, Archiving 및 이전 결과 사용
  - 모델 예측 결과가 기준치 초과할 경우, 새 결과 적용

- 그 외 분기가 필요한 작업


위와 같이 분기가 필요한 상황에 `BranchPythonOperator`를 이용할 수 있습니다.
해당 오퍼레이터를 사용하면 다음과 비슷한 workflow 모습을 볼 수 있습니다.

![img](/assets/img/post/airflow/branch.png)
*path_A가 선택되어 B 방향은 skip 된 모습*

첫 번째 Task가 완료되면 `check_situation`에서 적절한 path를 선택하여 작업 흐름을 이어나갈 수 있습니다. 선택되지 않은 path는 skip 처리되며 작업 흐름에 영향을 주지 않습니다.

- *`next_job`의 trigger_rule을 적절하게 바꿔 주어야 합니다.*
- *혹시 path_A에서 아무것도 하지 않더라도 operator가 있어야 다음 task가 동작합니다*


```python
#branch.py
import airflow
from airflow.models import DAG
from airflow.operators.dummy_operator import DummyOperator
from airflow.operators.python_operator import BranchPythonOperator

args = {
    'owner': 'mskim',
    'start_date': airflow.utils.dates.days_ago(2),
}

dag = DAG(
    dag_id='test_branch',
    default_args=args,
    schedule_interval="@daily",
    )

first_job = DummyOperator(
    task_id='first_job',
    dag=dag,
    )

options = ['path_A', 'path_B']

def which_path():
  '''
  return the task_id which to be executed
  '''
  if True:
    task_id = 'path_A'
  else:
    task_id = 'path_B'
  return task_id

check_situation = BranchPythonOperator(
    task_id='check_situation',
    python_callable=which_path,
    dag=dag,
    )

first_job >> check_situation

next_job = DummyOperator(
    task_id='next_job',
    trigger_rule='one_success', ## 중요! default 값은 'all_success' 입니다
    dag=dag,
    )


for option in options:
    t = DummyOperator(
        task_id=option,
        dag=dag,
        )
    if option == 'path_B':
        dummy_follow = DummyOperator(
            task_id='follow_' + option,
            dag=dag,
			)
        check_situation >> t >> dummy_follow >> next_job
    else:
        check_situation >> t >> next_job
```
`which_path` 함수가 들어간 자리를 상황에 맞게 바꾸면 됩니다.



### trigger_rule
트리거룰 옵션은 다음과 같습니다. 이중 all_success가 디폴트!
```python
ALL_SUCCESS = 'all_success'
ALL_FAILED = 'all_failed'
ALL_DONE = 'all_done' # 작업 성공 여부에 관계없이 모두 작동한 경우
ONE_SUCCESS = 'one_success'
ONE_FAILED = 'one_failed'
DUMMY = 'dummy'
NONE_FAILED = 'none_failed'
```

---

적어놓고 보니, 만약 선행 작업의 실패, 성공에 대응하려는 경우에는
trigger_rule만 조절해도 되겠네요.

- all_success(첫 작업)
  - all_success(성공시 작업)
  - all_failed(실패시 작업)
- one_success(다음 작업)
