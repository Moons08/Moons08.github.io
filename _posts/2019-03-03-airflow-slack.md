---
title: \[Airflow] Slack으로 결과 전달하기
date: 2019-03-03
tags: airflow
category: programming
---

작업 상황, 결과 등을 슬랙으로 전달하는 데 이용할 수 있는 Operator 입니다. (Mattermost도 됩니다.)

슬랙 토큰은 [여기](https://api.slack.com/custom-integrations/legacy-tokens)에서 생성하시면 됩니다. 슬랙 알림을 사용할 workspace를 선택해주세요.

예제를 실행하기 전 다음 명령어를 cmd 창에서 실행해 주세요.
**`pip install apache-airflow[slack]`**

```python
from datetime import timedelta
import airflow
from airflow import DAG
from airflow.operators.slack_operator import SlackAPIPostOperator

default_args = {
    'owner': 'mskim',
    'depends_on_past': False,
    'start_date': airflow.utils.dates.days_ago(2),
}

dag = DAG(
    'test_slack',
    default_args=default_args,
    schedule_interval='5 * * * *',
)

t1 = SlackAPIPostOperator(
  task_id='send_slack',
  token='Slack-Token', # 본인의 슬랙 토큰을 넣으시면 됩니다.
  channel='#genenral',
  username='Airflow',
  text='Hi. I am from Airflow! \n'
)

```
위 스크립트를 에어플로우 홈 아래의 dags 폴더에 넣으면 끝입니다.

이제 test를 해봅시다.

```bash
#!/bin/bash
airflow test test_slak send_slack 0 # 0만 입력하면 오늘 0시 0분으로 입력이 됩니다.
```

![img](/assets/img/airflow/slack.png)

잘 오네요!
토큰 생성하고 바로 실행하면 요청을 처리하지 못할 수 있습니다. 잠시 기다리시면 됩니다.

<br>

## 한걸음 더

### 접속정보 저장

매번 슬랙 토큰을 스크립트에 입력할 수도 있지만, webserver에 저장해두고 불러와서 사용할 수 있습니다.

http://localhost:8080/admin/connection/ 로 접속해서, `create`를 눌러봅시다.

![img](/assets/img/airflow/slack2.png)

*여기서 Login 항목은 사실 없어도 됩니다.*

Conn Id는 dag 스크립트에서 불러올 이름이 되고, Password에 Slack Token을 입력해두시면 됩니다.


### 알림 함수 설정하기

알림 하나 할 때마다 Operator를 만들 수는 없습니다. callback을 이용하면 편리합니다.
방금 저장한 접속 정보를 이용해서 클래스로 만들어 사용합시다.

- `alert.py`

```python
from airflow.hooks.base_hook import BaseHook
from airflow.operators.slack_operator import SlackAPIPostOperator

class SlackAlert:
    def __init__(self, channel):
        self.slack_channel = channel
        self.slack_token = BaseHook.get_connection('slack').password

    def slack_fail_alert(self, context):
        alert = SlackAPIPostOperator(
            task_id='slack_failed',
            channel=self.slack_channel,
            token=self.slack_token,
            text="""
                :red_circle: Task Failed.
                *Task*: {task}  
                *Dag*: {dag}
                *Execution Time*: {exec_date}  
                *Log Url*: {log_url}
                """.format(
                    task=context.get('task_instance').task_id,
                    dag=context.get('task_instance').dag_id,
                    exec_date=context.get('execution_date'),
                    log_url=context.get('task_instance').log_url,
                    )
                  )
        return alert.execute(context=context)

```
저는 위 스크립트를 dags 폴더 아래 utils 폴더에 넣어주었습니다.
이렇게 만든 클래스의 메소드를 callback 함수로 사용합니다.
airflow의 callback은 context라는 parameter를 전달하는데, 이에 대해서는 밑에서 설명하겠습니다.

- `test_slack.py`

```python
from utils.alert import SlackAlert # 그래서 이렇게 불러옵니다.
from datetime import timedelta
import airflow
from airflow import DAG
from airflow.operators.bash_operator import BashOperator

alert = SlackAlert('#airflow_test') # 메세지를 보낼 슬랙 채널명을 파라미터로 넣어줍니다.

default_args = {
    'owner': 'mskim',
    'depends_on_past': False,
    'start_date': airflow.utils.dates.days_ago(2),
    'on_success_callback': alert.slack_fail_alert,
    # 'on_failure_callback': alert.slack_fail_alert
}

dag = DAG(
    'test_slack',
    default_args=default_args,
    schedule_interval='0 9 * * *',
)

t1 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag,
)
```
*실제 사용할때는 `on_failure_callback`에 적용하면 됩니다.*

print_date 작업이 성공하면 다음과 같이 만들어둔 slack_fail_alert가 동작할 겁니다.

<br>

![img](/assets/img/airflow/slack03.png)

Log Url로 들어가면 작업 로그를 확인할 수 있습니다.

![img](/assets/img/airflow/slack04.png)

*airflow test 명령어로는 slack 메세지는 전송되지만 log가 생성되지 않습니다.*

<br>

### callback 함수의 context

콜백 함수에는 각 테스크의 상태정보가 dict타입으로 전달됩니다.

context의 인자들은 `airflow/models/__init__.py` 의 `get_template_context`에서 확인할 수 있습니다.

```python
return {
    'dag': task.dag,
    'ds': ds,
    'next_ds': next_ds,
    'next_ds_nodash': next_ds_nodash,
    'prev_ds': prev_ds,
    'prev_ds_nodash': prev_ds_nodash,
    'ds_nodash': ds_nodash,
    'ts': ts,
    'ts_nodash': ts_nodash,
    'ts_nodash_with_tz': ts_nodash_with_tz,
    'yesterday_ds': yesterday_ds,
    'yesterday_ds_nodash': yesterday_ds_nodash,
    'tomorrow_ds': tomorrow_ds,
    'tomorrow_ds_nodash': tomorrow_ds_nodash,
    'END_DATE': ds,
    'end_date': ds,
    'dag_run': dag_run,
    'run_id': run_id,
    'execution_date': self.execution_date,
    'prev_execution_date': prev_execution_date,
    'next_execution_date': next_execution_date,
    'latest_date': ds,
    'macros': macros,
    'params': params,
    'tables': tables,
    'task': task,
    'task_instance': self,
    'ti': self,
    'task_instance_key_str': ti_key_str,
    'conf': configuration,
    'test_mode': self.test_mode,
    'var': {
        'value': VariableAccessor(),
        'json': VariableJsonAccessor()
    },
    'inlets': task.inlets,
    'outlets': task.outlets,
}
```

각 task와 task_instance의 인자들을 알고 싶다면 다음 방법으로 확인!

```
vars(context.get('task'))
vars(context.get('task_instance'))
```

---

여기에서는 간단한 메세지를 보내는 기능이지만, 슬랙 API를 활용하면 더 많은 것들을 할 수 있을 겁니다.
