---
title: \[Airflow] XCom Tutorial 
date: 2020-07-18
tags: airflow 
category: programming
sidebar:
    nav: "airflow"
---

Airflow의 task는 독립적으로 실행되기 때문에 기본적으로는 서로 통신할 수단이 없습니다.
하지만 막상 작업 흐름을 만들다 보면 이전 작업의 결과, 요소 등을 다음 작업에 전달하면 깔끔하게 진행되는 경우가 있습니다.
그런 부분을 해결하기 위해 XCom을 이용해 메세지를 교환할 수 있습니다.

XCom은 cross communication의 줄임이라고 합니다. 간단한 예제와 함께 알아보겠습니다.

## XCom Example

> 주의: [`MAX XCOM Size is 48KB`](https://github.com/apache/airflow/blob/5c895154ed1d5df4adcdaea06ba65b8153148047/airflow/models/xcom.py#L39)  
> xcom은 task간의 통신을 위한 메모 정도의 목적으로 설계되어있기 때문에 대용량 파일 전송 등의 용도로는 적합하지 않습니다.

### dag arguments

```python
default_args = {
    "owner": "mk",
    "depends_on_past": False,
    "start_date": datetime(2020, 7, 1),
    "provide_context":True, # 주의
}

dag = DAG("xcom_tutorial",
        default_args=default_args,
        schedule_interval="@once",
        )
```

XCom을 적용하기 위해서는 `"provide_context":True` 설정이 꼭 있어야합니다. 개별 task에 설정해줄 수도 있고, 위처럼 dag 기본값으로 설정할 수도 있습니다.

### push

```python
def push_function(**context):
    return 'xcom_test'

def push_by_xcom_push(**context):
    context['task_instance'].xcom_push(key='pushed_value', value='xcom_push')

push_info = PythonOperator(
    task_id='push_info',
    python_callable=push_function,
    dag=dag,
)

push_by_xcom = PythonOperator(
    task_id='push_by_xcom',
    python_callable=push_by_xcom_push,
    dag=dag,
)
```

첫번째 예처럼 원하는 결과값을 바로 리턴할 수도 있고, 두번째 방식으로 키-값을 지정해서 `xcom_push()` 메소드로 전달할 수도 있습니다.
PythonOperator의 첫번째 예처럼, 다른 오퍼레이터의 `execute()` 메서드가 실행되어 값을 리턴하게 되면 그 값도 xcom에 자동으로 푸쉬된다고 합니다.

### pull

```python
def pull_function(**context):
    # ti는 task_instance의 줄임
    value = context['ti'].xcom_pull(task_ids='push_info')
    print(value)

pull_1 = PythonOperator(
    task_id='pull_info_1',
    python_callable=pull_function,
    dag=dag,
)
{% raw %}
pull_2 = BashOperator(
    task_id='pull_info_2',
    bash_command='echo "{{ ti.xcom_pull(key="pushed_value") }}"', # .sh 파일 안에서도 사용 가능!
    dag=dag,
)
{% endraw %}
```

이렇게 xcom으로 보낸 정보를 xcom_pull로 받아올 수가 있는데, dag내 task뿐만 아니라 task가 실행하는 스크립트에서도 jinja template을 이용해 사용할 수 있습니다.

### with Subdag

subdag로 자주 사용되는 프로세스를 만들어 두고 사용하는 경우도 있겠죠.

```python
def xcom_subdag(parent_dag_name, child_dag_name, args):

    def pull_and_push(**context):
        values = context['ti'].xcom_pull(dag_id=parent_dag_name) # pull from parent
        val1 = context['ti'].xcom_pull(dag_id=parent_dag_name, task_ids='push_info')
        val2 = context['ti'].xcom_pull(dag_id=parent_dag_name,
                                        task_ids='push_by_xcom', key='pushed_value')

        context['ti'].xcom_push(key='val0', value=values)# push to child
        context['ti'].xcom_push(key='val1', value=val1)
        context['ti'].xcom_push(key='val2', value=val2)

        print(values)
        return values

    dag = DAG(
        dag_id='%s.%s' % (parent_dag_name, child_dag_name),
        default_args=args,
        schedule_interval=None,
    )

    pull_from_parent = PythonOperator(
        task_id='%s-pull_from_parent' % (child_dag_name),
        python_callable=pull_and_push,
        dag=dag,
        )
    return dag

```

위처럼 값을 받아올 dag_id를 지정할 수가 있습니다. key로 값을 전달한 경우에는 key도 꼭 명시를 해주어야합니다.

![img](/assets/img/post/airflow/xcom_subdag.png)
*WebUI로 확인한 Subdag의 XCom 결과*

여러개 전달하고 dag_id만 던져주면 어떻게 하나 했더니 저렇게 가져오네요. `(val0, return_value)`

```python
def xcom_subdag(parent_dag_name, child_dag_name, args, params=None):

    def pull_function(params, **context):
        values = params if params else context['ti'].xcom_pull(dag_id=parent_dag_name)
```

혹은 위처럼 xcom을 사용하지 않고 subdag를 만들때 추가 argument를 전달할 수도 있습니다.

---

참고: [Airflow Concepts - Xcom](https://airflow.apache.org/docs/stable/concepts.html#xcoms)
