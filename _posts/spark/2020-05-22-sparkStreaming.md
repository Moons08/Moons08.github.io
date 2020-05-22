---
title: Spark로 실시간 데이터 처리하기 (Intro)
date: 2020-05-22
tags: Spark
category: programming
toc: True
header:
  teaser: /assets/img/post/spark/streaming-arch.png
sidebar:
    nav: "spark"
---

실시간으로 입수되는 데이터를 Spark로 처리하는 방법에 대해 정리합니다. 스파크 2.2버전 기준으로 작성되었습니다. ~~스파크 3.0의 정식 릴리즈를 기다리며~~

## Intro

![img](/assets/img/post/spark/streaming-arch.png)

Spark Streaming을 사용해서 HDFS/S3로 표현된 File (parquet, json, orc, csv 등) 혹은 Kafka같은 Pub/Sub 소스에서 데이터를 읽어와서 원하는 방식으로 데이터를 처리할 수 있습니다. 이 외에 테스트용 소켓 소스도 지원합니다.

## Spark Streaming API

스파크에는 2가지 방식의 스트리밍 처리 API가 있습니다. 바로 DStream과 구조적 스트리밍(Structured Streaming)입니다. DStream이 먼저 나왔고, 이후에 최적화 기술을 지원하는 구조적 스트리밍이 추가되었습니다. 구조적 스트리밍은 그 이름처럼 스파크의 구조적 API(DataFrame, Dataset, SQL)를 지원하기 때문에 기존 배치성으로 작성된 코드를 대부분 가져다 쓸 수 있습니다. RDD보다는 DataFrame을 선호하는 것처럼, Structured Streaming에 먼저 익숙해져보겠습니다.

|특징|DStream|Structured Streaming|
|:--:|:--:|:--:|
|구조적 테이블|미지원|DataFrame/DataSet API 지원|
|이벤트 시간 처리|미지원|지원|
|연속형처리|지원|2.3이후 버전부터 지원[Testing]|

## Simple Example

아래 예제 코드는 파일을 읽어서 간단한 집계 작업을 한 뒤, 결과를 디버깅용 memory table (temp view!)에 저장합니다.

**code first!**

```scala
//scala
val data = spark.read.parquet("/data/stream")
val dataSchema = data.schema

// read
val streaming = spark.readStream
                .schema(dataSchema)
                .option("maxFilesPerTrigger", 1)
                .parquet("/data/stream")
// transform
val ageCounts = streaming.groupBy("age").count()

// write
val ageQuery = ageCounts
                .writeStream
                .queryName("age_counts")
                .format("memory").outputMode("complete")
                .start()

ageQuery.awaitTermination() // 끝
```

```python
#python
data = spark.read.parquet("/data/stream")
dataSchema = data.schema

streaming = spark.readStream\
                .schema(dataSchema)\
                .option("maxFilesPerTrigger", 1)\
                .parquet("/data/stream")

ageCounts = streaming.groupBy("age").count()
ageQuery = ageCounts\
                .writeStream\
                .queryName("age_counts")\
                .format("memory").outputMode("complete")\
                .start()

ageQuery.awaitTermination() # 끝
```

설명은 scala 기준으로 하지만 python 코드도 거의 동일합니다.

```scala
val data = spark.read.parquet("/data/stream")
val dataSchema = data.schema
```

스트림 데이터의 형식이 언제 어떻게 바뀔지 모르니 미리 정해둔 스키마로 읽어옵니다. 여기서는 동일 파일의 schema를 이용합니다.

### read

```scala
val streaming = spark.readStream // read -> readStream
                .schema(dataSchema)
                .option("maxFilesPerTrigger", 1) // 한번에 읽을 최대 파일 수
                .parquet("/data/stream")
```

[Data ETL with Spark](/programming/dataETL/)에서 본 Spark API 형식이 여기에도 적용됩니다. `read`만 `readStream`로 바꿔주고, 스트림 데이터 특성에 맞는 옵션을 설정하면 됩니다.

### write

```scala
val ageCounts = streaming.groupBy("age").count() // age별로 묶어서 count!
val ageQuery = ageCounts
                .writeStream
                .queryName("age_counts") // 결과 테이블명
                .format("memory") // 여기서는 메모리에 저장합니다.
                .outputMode("complete") // append, update 도 있습니다.
                .start()
```

스트림도 배치 스파크 API처럼 지연 처리(lazy evaluation) 방식으로 동작합니다.
액션을 일으키는 부분 `start()` 전까지는 실행 계획만 세워두고, 액션 이후에 실제로 작업을 시작합니다.
이 코드는 강제로 종료하기 전까지 계속 해당 경로의 데이터를 읽고 처리하여 메모리에 올리는 작업을 합니다.

### awaitTermination

```scala
ageQuery.awaitTermination() // 실행 중인 코드가 끝날 때까지 대기 (사실상 ctrl+c 누르기 전까지 무한..)
```

`awaitTermination`을 지정해 주지 않으면 도중에 프로세스가 종료되기 때문에 꼭 있어야합니다.

> 왜냐하면... 스트리밍 쿼리는 **독립적인** 데몬 쓰레드에서 동작하기 때문입니다. Java에서 데몬 쓰레드는 스파크 앱의 메인 쓰레드가 완료되기 전까지 병렬 처리를 수행하기 위해 사용되는데, 마지막 일반 쓰레드가 완료되고 나면, JVM은 전체 스파크 앱을 종료합니다. **그러니까 다른 데몬 쓰레드**(여기에서는 스트리밍 쿼리)**는 아직 끝나지도 않았는데 메인 쓰레드가 지 할 거 다 끝났다고 문닫고 집에가기 때문에** 붙잡기 위해 `awaitTermination`을 지정해 주어야 합니다.  
>
> 데몬: 사용자가 직접적으로 제어하지 않고, 백그라운드에서 돌면서 여러 작업을 하는 프로그램을 말합니다.

### zeppelin settting

코드를 실행하시기 전에 만약 저와 같은 제플린-기본 도커 이미지 환경에서 테스트 하신다면, 제플린에서 다음 환경설정을 먼저 해주세요.

![img](/assets/img/post/spark/Streaming/zeppelin_setting.png)
*우측 상단의 anonymous라 표시된 부분(유저명) 클릭 후 -> Interpreter 클릭*

![img](/assets/img/post/spark/Streaming/spark_interpreter.png)
*그리고 많고 많은 인터프리터들 중 스파크를 찾아 `Globally` 를 `Per note`로 바꿔줍니다. 이제 인터프리터를 노트북마다 개별로 사용할 수 있습니다. globally 상태에서는 노트북 하나 돌고 있으면 끝날때까지 다른 애들은 무한 Pending 상태가 됩니다.*

### execute

위 설정을 마치고 나서 한 노트북에서는 위의 코드를 실행하고, 다른 노트북을 열어서 아래 코드로 메모리에 올려놓은 테이블을 조회해봅시다.
아래 코드는 스트림 코드가 도는 동안 1초마다 결과 테이블을 조회합니다. 값들이 계속 바뀌는 것을 확인할 수 있습니다.
조회 전에 다 끝내버릴 수 있으니 실시간으로 변하는 과정을 보고 싶다면 더 많은 데이터로 테스트 하시거나 손 스피드를 높여주세요.

```scala
// spark.streams.active // 실행 중인 스트림 목록 확인용
// scala
for (i <- 1 to 5) {
    spark.sql("SELECT * FROM age_counts").show()
    Thread.sleep(1000) // 1초
}
```

```python
# python
import time

for x in range(5):
    spark.sql("SELECT * FROM age_counts").show()
    time.sleep(1)

# 결과
+-----------+-----+
|        age|count|
+-----------+-----+
|  75+ years| 2668|
| 5-14 years| 2824|
|25-34 years| 2781|
|35-54 years| 2806|
|15-24 years| 2837|
|55-74 years| 2776|
+-----------+-----+

+-----------+-----+
|        age|count|
+-----------+-----+
|  75+ years| 3188|
| 5-14 years| 3269|
|25-34 years| 3248|
|35-54 years| 3261|
|15-24 years| 3285|
|55-74 years| 3223|
+-----------+-----+

+-----------+-----+
|        age|count|
+-----------+-----+
|  75+ years| 3709|
| 5-14 years| 3689|
|25-34 years| 3719|
|35-54 years| 3724|
|15-24 years| 3739|
|55-74 years| 3676|
+-----------+-----+

+-----------+-----+
|        age|count|
+-----------+-----+
|  75+ years| 4642|
| 5-14 years| 4610|
|25-34 years| 4642|
|35-54 years| 4642|
|15-24 years| 4642|
|55-74 years| 4642|
+-----------+-----+

+-----------+-----+
|        age|count|
+-----------+-----+
|  75+ years| 4642|
| 5-14 years| 4610|
|25-34 years| 4642|
|35-54 years| 4642|
|15-24 years| 4642|
|55-74 years| 4642|
+-----------+-----+

```

조회하는 코드는 다 돌았는데, 스트리밍 코드는 계속 돌고 있다구요? 원래 그런 코드입니다. 새로운 데이터가 입수되면 끊임없이 처리해야하니까요.
~~제플린에서 cancel 버튼을 아무리 눌러도 멈추지 않는 것은 함정... 저는 도커를 내렸다 올렸습니다.~~
테스트시 이런 상황을 방지하기 위해 Trigger를 사용할 수 있습니다.

## Structured Streaming

구조적 스트리밍은 스트림 데이터를 지속적으로 추가되는 구조화된 테이블(데이터프레임 혹은 데이터셋)처럼 다룹니다. 그래서 (일부 제약이 있긴 하지만) 대부분의 배치성 코드를 가져다 쓸 수 있습니다. 스트리밍 처리에 필요한 내고장성, 최적화, 정확히 한번 처리 등은 스파크가 해줍니다.

### source (read)

데이터 소스로는 아파치 카프카, HDFS, S3, (테스트용) 소켓 소스를 지원합니다.
파일 소스를 사용할 때는 위의 예제처럼 `maxFilesPerTrigger`를 이용해서 한번에 읽을 파일 수를 결정할 수 있습니다.

kafka 소스를 쓸 때는 몇가지 옵션이 더 있습니다. ~~저는 카프카가 분산처리 지원하는 Pub/Sub 구조인 것 정도 밖에 모릅니다.. 다음편을 쓰며 보완해보겠습니다.~~

* assign - 토픽 & 카프카 파티션 지정
* subscribe - 토픽 목록 지정
* subscribePattern - 토픽 목록 & 패턴 지정

```scala
//scala
val ds = spark.readStream.format("kafka")
          .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
          .option("subscribe", "topic1") // 토픽 하나
          // .option("subscribe", "topic1,topic2") // 토픽 여러개
          // .option("subscribePattern", "topic.*") // 토픽 여러개 with Pattern
          .load()


```

### sink (write)

싱크로 (*하수구에 물 버리듯?*) 결과 데이터의 목적지를 지정합니다.
소스와 비슷하게 카프카, 파일을 지원하고, 테스트와 디버깅용으로 콘솔, 메모리로도 싱크할 수 있습니다.

배치 작업의 `write.mode` 옵션처럼 출력 모드를 설정할 수 있습니다. 무려 업데이트가 됩니다.
다만, 출력 모드마다 제약 조건이 있습니다.

* append: 신규 데이터 추가
* update: 변경 데이터 갱신
* complete: 전체 출력 재작성

### Trigger

트리거를 통해 `언제` 출력할지 결정할 수 있습니다. 기본적으로는 이전 작업이 끝나면 바로 다음 작업에 들어가지만,
너무 빠른 속도로 끝없이 동작하면 부하가 많이 가기 때문에 처리 시간 기반으로 (100초마다 등) 설정하거나, 단 한번만 작동하도록 설정할 수 있습니다.

* [Spark로 실시간 데이터 처리하기 (Trigger)](/programming/streamTrigger)

### Event Time

스트리밍 데이터는 그 특성상(거리, 통신상태 등의 변수) 데이터 생성 시점과 도착 시점이 다를 수 있습니다.
구조적 스트리밍은 타임스탬프를 이용해 도착 시점 기준이 아닌 생성시점 기준으로도 처리를 지원합니다.

### CheckPoint

체크포인트 경로를 설정해서 장애시에도 그 부분부터 스트림 처리를 다시 시작할 수 있습니다.

## Outro

이번 편은 간단하게 맛보기 정도로 작성하고, 다음 편부터 구조적 스트리밍을 중심으로 각 요소마다 자세히 다뤄보겠습니다.
