---
title: \[Airflow] Jinja Template 써보기
date: 2019-03-16
tags: airflow
category: programming
---

Airflow는 flask에서 자주 사용되는 강력한 도구인 Jinja Template을 지원합니다.


```python
templated_command = """
{% raw %}
{% for i in range(5) %}
    echo "{{ ds }}"
    echo "{{ macros.ds_add(ds, 7)}}"
    echo "{{ params.my_param }}"
{% endfor %}
{% endraw %}
"""
```
airflow tutorial을 설명하며 위와 같은 코드를 봤습니다.

굉장히 익숙치 않은 `{% raw %}{%{% endraw %}` 기호입니다.
저 기호를 이용해서 jinja template 안의 if, for문 등을 제어할 수 있습니다.
템플릿은 dag파일에 있어도 되고, task가 실행할 파일(sh, scala, py, ...) 안에 있어도 동작합니다.


다음 코드는 BashOperator를 이용해서 템플릿이 적용된 shell script를 실행하는 예제입니다.

```python
# jinja.py
from datetime import timedelta
import airflow
from airflow import DAG
from airflow.operators.bash_operator import BashOperator

default_args = {
    'owner': 'mskim',
    'start_date': airflow.utils.dates.days_ago(2),
}

dag = DAG(
    'test_jinja',
    default_args=default_args,
    schedule_interval='5 0 * * *',
    template_searchpath=['/home/mk/airflow/scripts'],
    # 위 경로에서 실행할 script를 찾습니다.
)

test1 = "It would be passed, but jinja does not work {{ ds }}"

t = BashOperator(
      task_id='test',
      bash_command='jinja.sh',
      params={'test':test1},
      dag=dag,
      )
```
`test`라는 Task에서 `jinja.sh` 파일을 실행시키고, 미리 설정해둔 test1 파라미터를 전달합니다.

다음은 위에서 전달한 파일 내용입니다.

```sh
# jinja.sh
#!/bin/bash

# dag 정보와 task 정보도 Jinja template으로 불러올 수 있습니다.
echo "{{ dag }}, {{ task }}"

# dag file에서 전달해 준 test 파라미터는 다음과 같이 쓸 수 있습니다.
echo "{{ params.test }}"

# execution_date를 표현합니다.
echo "execution_date : {{ execution_date }}"
echo "ds: {{ds}}"

# macros를 이용하여 format과 날짜 조작이 가능합니다.
echo "the day after tommorow with no dash format"
echo "{{ macros.ds_format(macros.ds_add(ds, days=2),'%Y-%m-%d', '%Y%m%d') }}"
{% raw %}
# for, if 예제입니다.
{% for i in range(3) %}
	echo "{{ i }} {{ ds }}"
  {% if i%2 == 0  %}
		echo "{{ ds_nodash }}"
  {% endif %}
{% endfor %}
{% endraw %}
```

그럼 test 명령을 이용해 확인해 보겠습니다.
`airflow test test_jinja test 0`


```sh
#result
[2019-03-16 14:30:12,525] {bash_operator.py:119} INFO - Output:
[2019-03-16 14:30:12,527] {bash_operator.py:123} INFO - <DAG: test_jinja>, <Task(BashOperator): test>
[2019-03-16 14:30:12,527] {bash_operator.py:123} INFO - It would be passed, but jinja does not work like this {{ ds }}
[2019-03-16 14:30:12,527] {bash_operator.py:123} INFO - execution_date : 2019-03-16T00:00:00+00:00
[2019-03-16 14:30:12,527] {bash_operator.py:123} INFO - ds: 2019-03-16
[2019-03-16 14:30:12,527] {bash_operator.py:123} INFO - the day after tommorow with no dash format
[2019-03-16 14:30:12,527] {bash_operator.py:123} INFO - 20190318
[2019-03-16 14:30:12,527] {bash_operator.py:123} INFO - 0 2019-03-16
[2019-03-16 14:30:12,527] {bash_operator.py:123} INFO - 20190316
[2019-03-16 14:30:12,527] {bash_operator.py:123} INFO - 1 2019-03-16
[2019-03-16 14:30:12,527] {bash_operator.py:123} INFO - 2 2019-03-16
[2019-03-16 14:30:12,528] {bash_operator.py:123} INFO - 20190316
[2019-03-16 14:30:12,528] {bash_operator.py:127} INFO - Command exited with return code 0

```


Jinja Template을 이용하면 반복적인 작업(대부분의 에어플로우 잡이겠죠)의 날짜 정보 전달, 조작이 간편해집니다.
특히 과거 데이터를 한꺼번에 작업해야할 경우 execution_date를 기준으로 작업을 하면 훨씬 관리가 편해집니다.
*제 경우에는 쿼리에 작업연월, 전월, 차월 등을 입력해야 하는 부분이 몇 군데 있었는데, 많은 도움이 되었습니다*

BashOperator와 쉘 스크립트로 예제를 만들었지만, Python, Oracle, SSH등 다른 Operator들도 동일한 방법이 가능합니다.
