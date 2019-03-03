---
title: [Airflow] Basic Concept 알아보기
date: 2019-03-02
tags: airflow
category: programming
---
에어플로우를 사용하기 위한 가장 기초적인 개념에 대해 정리해봤습니다.

에어플로우는 workflow에 대해 설명하고, 실행하고, 모니터링하는 플랫폼 도구입니다.
>The Airflow Platform is a tool for describing, executing, and monitoring workflows. - Airflow Official Documents

사실 두가지만 알면 됩니다. DAG & Operator.
DAG와 Operator를 결합하여 Task Instance를 만들면 복잡한 Workflow를 만들 수 있습니다.

- DAG : 작업이 수행되어야 하는 순서에 대한 설명
- Operator : 어떤 작업을 수행하기 위한 템플릿으로 작동하는 클래스
  - Task : Operator의 매개 변수화 된 인스턴스
    - Task Instance : 1) DAG에 할당되고 2) DAG의 특정 실행과 관련된 상태가 있는 Task

*아래는 좀 더 깊게 설명한 airflow 공식 문서를 조금 변형하여 가져왔습니다.*

<br>

## DAGs

Airflow에서 DAG 또는 Directed Acyclic Graph(비 순환 그래프)는 실행하려는 모든 작업의 ​​모음으로, 각 작업의 관계 및 종속성을 표현합니다. *비 순환이란, 말 그대로 순환하지 않음을 뜻합니다.  A - B - A - C의 workflow는 동작하지 않습니다. A - B - A' - C는 동작합니다.*

예를 들어, 다음 규칙들을 따르는 간단한 DAG를 설정할 수 있습니다.

- A, B, C 세 가지 작업이 있습니다.
- A가 성공적으로 실행되어야만 B가 실행되고, C는 A와 무관하게 항상 실행되어야 합니다.
- A 작업이 시작된 지 5분이 지나면 자동으로 종료되어야 합니다.
- B 작업은 실패할 경우 5 번까지 다시 시작되어야 합니다.
- workflow는 매일 밤 10시에 실행되어야 하고, 특정 날짜 이전에는 시작하지 않습니다.

위와 같이 DAG는 workflow의 수행 방법을 설명합니다. 하지만 DAG는 A, B, C가 어떤 작업인지 알 수 없고, 알아야 할 필요도 없습니다. 가령 A가 B에서 분석 할 데이터를 준비하는 동안 C가 이메일을 보내는 작업일 수도 있고, 또는 A가 당신의 위치를 확인하여 B가 차고 문을 여는 동안 C가 집의 전등을 켜는 작업일 수 도 있습니다. 중요한 것은 DAG가 각 작업이 하는 일과 관련이 없다는 것입니다. **DAG의 임무는 그들이 하는 일이 적시에, 올바른 순서로, 또는 예기치 않은 문제의 올바른 처리로 발생하는지 확인하는 것입니다.**

DAG는 Airflow의 `DAG_FOLDER`에있는 표준 Python 파일에 정의되어 있습니다. Airflow는 각 파일의 코드를 실행하여 DAG 개체를 동적으로 작성합니다. 원하는 수의 DAG를 가질 수 있으며 각 DAG는 임의의 수의 작업을 설명합니다. 일반적으로 DAG는 각각 하나의 논리적 workflow에 대응해야 합니다.


#### Default Arguments
예제에서 봤던 그것입니다. 미리 dag에 전달할 *(최종적으로는 Operator들에게 전달될)* arguments들을 정의해 놓으면 편리하게 사용할 수 있습니다. 자주 쓰는 몇 가지만 설명하도록 하겠습니다. 전체 파라미터 설명은 [여기](https://airflow.apache.org/code.html#airflow.models.BaseOperator)에서 확인할 수 있습니다.

```python
default_args = {
    'owner': 'Airflow' # 각 테스크의 owner. linux username이 추천됨
    'start_date': datetime(2016, 1, 1), # 작업이 시작될 날짜
    'end_date': datetime(2016, 2, 1), # 이 날짜 이후로는 작업을 시행하지 않음
    'retries': 1, # 최대 재시도 횟수
    'retry_delay': timedelta(minutes=5), # 재시도 간 딜레이
    'depends_on_past': False, # true일 경우, 이전 분기 작업이 성공해야만 작업을 진행
    'on_failure_callback':some_function(), # task가 실패했을 경우 호출할 함수, dictype의 context를 전달.
    'on_retry_callback':some_function2(), # 재시도시, 상동
    'on_success_callback':some_function3(), # 성공시, 상동
    'priority_weight': 10, # 이 테스크의 우선순위 가중치, 높을 수록 먼저 triggered
}

dag = DAG('my_dag', default_args=default_args)
op = DummyOperator(task_id='dummy', dag=dag)
print(op.owner) # Airflow
```

#### Scope

에어플로우는 DAGfile 안의 모든 `DAG` 오브젝트들을 불러올 수 있습니다. 다만, 각 오브젝트들은 전역 변수`globals()`여야 합니다. 아래의 예에서 `dag_1`만 불려오게 될 것입니다.

```python
dag_1 = DAG('this_dag_will_be_discovered')

def my_function():
    dag_2 = DAG('but_this_dag_will_not')

my_function()
```
이런 성질은 유용하게 쓰일 곳이 있을 겁니다. `SubDagOperator`를 확인해보세요.

<br>

## Operator
**DAG는 workflow를 실행하는 방법을 설명하지만, Operator는 실제로 수행할 작업을 결정합니다.**

Operator는 대개 (항상 그런 것은 아니지만) 독립적으로 동작합니다. 따라서 서로 다른 Operator 간에 자원을 공유 할 필요가 없습니다. DAG는 Operator들이 올바른 순서로 운영되는지 확인합니다. 실제로 Operator들은 완전히 다른 기계에서 작동 할 수도 있습니다.

만약 두 개의 Operator가 파일 이름이나 소량의 데이터와 같은 정보를 공유해야 하는 경우에는 (가능하다면) 단일 Operator로 합치는 것이 나을 수 있습니다. 여의치 않은 경우에는 airflow가 지원하는 XCom 기능을 활용할 수 있습니다.

에어플로우는 다음 Operator들을 지원합니다.

- BashOperator - bash command 실행
- PythonOperator - 임의의 파이썬 함수 호출
- EmailOperator - email 송신
- SimpleHttpOperator - HTTP request 송신
- MySqlOperator, SqliteOperator, PostgresOperator, MsSqlOperator, OracleOperator, JdbcOperator, etc. - SQL command 실행
- Sensor - 다음과 같은 것들을 기다리며 확인 (time, file, database row, S3 key, etc…)

위의 나열된 것 보다 **훨씬 많은** Operator들이 존재합니다. 이거 있나? 싶으면 보통 있습니다. 다만 contrib 디렉토리에 있는 Operator들은 community에 의해 작성된 것이므로 완벽하지 않을 수 있습니다.

#### Tasks

Operator가 인스턴스화되면 이를 `Task`라고 합니다. 매개 변수화 된 작업은 DAG의 노드가 됩니다.

#### Task Instance

Task Instance는 Task의 특정 실행을 나타내며 (일반적으로 여러 번 실행될 것이기 때문에) dag, task, 특정 시점의 조합으로 구분할 수 있습니다. Task Instance는 “running”, “success”, “failed”, “skipped”, “up for retry” 등의 표시 상태를 가집니다.

## Set streams

Operator 간의 순서를 정해주려면 다음과 같은 방법으로 표현할 수 있습니다.
1.8 버전부터 Bitshift Composition을 지원한다고 하네요.

```python
op1 >> op2
op1.set_downstream(op2) # 전통적 방법, op1이 완료된 뒤 op2가 실행됩니다.

op2 << op1
op2.set_upstream(op1)

# 양방향 지원이 된답니다.
op1 >> op2 >> op3 << op4
```

```python
# Dag를 지정하는데도 적용됩니다.
dag >> op1 >> op2

#위 코드는 아래 코드와 동일한 작업을 합니다.
op1.dag = dag
op1.set_downstream(op2)
```

---
이외에도 많은 구성 요소가 있지만, 예제를 진행하며 하나씩 풀어볼까 합니다.
