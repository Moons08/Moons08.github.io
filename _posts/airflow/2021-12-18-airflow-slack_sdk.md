---
title: Slack SDK로 메세지 보내기 (with Airflow)
date: 2021-12-18
tags: airflow sdk slack
category: programming
toc: true
sidebar:
    nav: "airflow"
---

Airflow 버전에 따라서 의존성 때문에 slack operator 쓰기가 영 불편합니다. 그래서 python slack sdk로 slack operator를 대체하는 방법에 대해 정리해봅니다.

> Q. 왜 Airflow가 지원하는 Operator를 쓰지 않나요?  
> A. 언제 지원이 끊길지 모르고, 버전별로 호환이 될지 모르기 떄문입니다. sdk를 사용하면 의존성 단계를 한단계 줄일 수 있습니다. (slack과 나 사이의 Airflow 의존성을 제거)

[python-slack-sdk github](https://github.com/slackapi/python-slack-sdk/tree/main/tutorial)를 참조하시면 더 자세한 튜토리얼을 확인할 수 있습니다.

## install

Python 3.6 이상만 지원한다고 합니다.

```sh
pip install slack_sdk
```

## Basic Usage

code first.

```python
from slack_sdk import WebClient

client = WebClient(token='SLACK_BOT_TOKEN')
response = client.chat_postMessage(channel='#random', text="Hello world!")
```

`WebClient` 클래스로 slack message를 보낼 `client` instance를 만들었습니다. 그리고 `chat_postMessage` 메서드로 보낼 채널과, 텍스트를 정해서 전달합니다. `response` 로는 전송 성공, 실패 결과가 전달됩니다. 

여기에 error 처리를 추가할 수도 있고, slack_sdk는 async 함수도 지원하기 때문에 필요하면 가져다 쓰면 되겠습니다.

> Q. 그런데 내가 사용하는 Slack 에 연결할 토큰은 어디서 찾을 수 있나요?  
> A. [여기](https://github.com/slackapi/python-slack-sdk/blob/main/tutorial/01-creating-the-slack-app.md)에 그림과 함께 설명되어 있습니다. SlackApp을 생성하고 권한을 부여해서 OAuth Access Token이라는 것을 발급해야 합니다. 


## with Airflow

airflow 에서는 간단하게는 아래처럼 사용할 수 있습니다.

```python
from datetime import datetime

from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from slack_sdk import WebClient

default_args = {
    'owner': 'mskim',
    'depends_on_past': False,
    'start_date': datetime(2021, 1, 1),
}

dag = DAG(
    'test_slack',
    default_args=default_args,
    schedule_interval='@once',
)


def send_slack_message(channel, message):
    client = WebClient(token='SLACK_BOT_TOKEN')
    client.chat_postMessage(channel='#random', text="Hello world!")


with dag:
    PythonOperator(
        task_id="send_slack_message",
        python_callable=send_slack_message,
        op_kwargs=dict(
            channel="#random",
            text="hello world!",
        ),
    )

```

airflow에서 자주 사용하는 callback 함수는 [[Airflow] Slack으로 결과 전달하기](/programming/airflow-slack/)을 참조해서 클래스를 만들거나, 함수를 반환하는 함수를 만들어서 사용할 수도 있겠습니다.


## closing

축하합니다! airflow 의존성 없이 python으로 언제든 slack message를 보낼 수 있게 되었습니다.
