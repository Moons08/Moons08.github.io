---
title: Spark로 실시간 데이터 처리하기 (Trigger)
date: 2020-05-22
tags: Spark
category: programming
toc: True
header:
  teaser: /assets/img/post/spark/streaming-arch.png
sidebar:
    nav: "spark"
---

구조적 스트리밍은 트리거를 통해 `언제` 출력할지 결정할 수 있습니다. 기본적으로는 이전 작업이 끝나면 (마이크로 배치) 바로 다음 작업에 들어가지만,
너무 빠른 속도로 끝없이 동작하면 부하가 많이 가기 때문에 인터벌(100초 간격 등)을 설정하거나, 단 한번만 작동하도록 설정할 수 있습니다.

## Fixed interval micro-batches

![img](/assets/img/post/spark/streaming-flow.png)

고정 간격 마이크로 배치. 즉, 고정된 인터벌(쉬는시간)을 주고 작업을 시킵니다. 만약 이전 작업이 오래 걸리는 경우에는 인터벌 없이 바로 다음 작업이 시작됩니다.
사용 가능한 새 데이터가 없으면 마이크로 배치가 시작되지 않습니다. (2.4.5 버전 기준)

```scala
import org.apache.spark.sql.streaming.Trigger

df.writeStream
  .format("console")
  .trigger(Trigger.ProcessingTime("100 seconds"))
  .start()
```

```python
# python
df.writeStream \
  .format("console") \
  .trigger(processingTime='100 seconds') \
  .start()
```

## One-time micro-batch

일회성 마이크로 배치. 사용 가능한 모든 데이터를 처리하고 자체적으로 중지하기 위해 단 한 번의 마이크로 배치를 실행합니다. 테스트할 때 유용하게 쓸 수 있고, 자주 실행되지 않는 (굳이 스트리밍을 계속 켜놓지 않아도 될)잡을 수동으로 실행할 때도 사용할 수 있습니다.

```scala
df.writeStream
  .format("console")
  .trigger(Trigger.Once())
  .start()
```

```python
# python
df.writeStream \
  .format("console") \
  .trigger(once=True) \
  .start()
```

## Continuous with fixed checkpoint interval (experimental)

기본적으로 스파크 스트리밍은 마이크로 배치 방식으로 작업을 합니다. 마이크로 배치 방식은 어느 정도 데이터가 쌓이길 기다려서 한꺼번에 효율적으로 작업을 할 수 있지만, 지연 시간이 발생한다는 단점이 있습니다. 반면, 연속형 처리는 레코드 하나 오면 하나씩 바로 작업하는 방식이기 때문에 굉장히 빠른 응답 속도를 보여줍니다. 하지만 당연히 부하가 커서 같은 데이터량을 처리하는데 드는 비용이 더 많이 듭니다.

정확히 한번 실행을 보장(exactly-once guarantees)하는 마이크로 배치 방식보다 100배나 빠르다고 하네요. 다만, 최소 1회(at-least-once guarantees)를 보장한다고 합니다. 중복이 있을 수 있다는 거겠죠?

연속처리모드는 스파크 2.3 버전부터 사용할 수 있습니다.

```scala
df.writeStream
  .format("console")
  .trigger(Trigger.Continuous("1 second"))
  .start()
```

```python
# python
df.writeStream
  .format("console")
  .trigger(continuous='1 second')
  .start()
```
