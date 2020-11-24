---
title: spark write parquet with null
date: 2020-11-24
tags: Spark
category: programming
sidebar:
    nav: "spark"
---

spark sql로 작업을 하다보면 auto schema 때문에(혹은 연산을 거친 후에) 형식이 바뀌는 경우가 발생하는데, 갑자기 튀어나오는 Null이 종종 문제가 됩니다.
파케이로 저장하면 Null 형식을 지원하지 않는다고 하면서 에러를 뿜거든요.

## 문제

Null을 그냥 집어넣으려고 하면 아래 같은 에러가 나옵니다.

```scala
spark.sql("select 'test' as a, null as b")
.write.parquet("test")

// org.apache.spark.SparkException: Job aborted.
// ...
// Caused by: java.lang.RuntimeException: Unsupported data type NullType.
```

스키마를 확인해보면 왜 이러는지 볼 수 있습니다.

```scala
spark.sql("select 'test' as a, null as b").printSchema()

// root
//  |-- a: string (nullable = false)
//  |-- b: null (nullable = true)
```

b 컬럼의 `타입`이 null로 들어가있습니다. 에러 메세지에서 보여준 것처럼 아쉽게도 null type을 지원하지 않습니다.
null type을 지원하지 않는 것이지 null 자체를 지원하지 않는 것은 아니기 때문에 당연히 null 값을 넣을 수 있습니다.

## 해결

cast로 형식을 지정해주면 됩니다.

```scala
spark.sql("select 'test' as a, cast(null as string) as b")
.write.parquet("test")
```

간단합니다.

csv처럼 write option에 schema 넣어서 해결해야지! 해봐야 parquet는 쓰기 옵션에 스키마를 지원하지 않습니다. (삽질 경험)

## +

null 대신 공백이나 대체 문자로 저장하고 쓰면 코드가 돌기는 합니다.
하지만 spark 뿐만 아니라 많은 언어에서 그럴텐데, 빈 값에 대해서는 Null로 확실하게 표현해줘야 최적화를 수행할 수 있습니다.

그리고 `printSchema`에서 `nullable=false` 여도 강제성은 없습니다.
nullable 속성은 스파크 옵티마이저에게 `여기 null 안들어가는 컬럼이야`라고 알려주는 정도의 기능을 한답니다.
옵티마이저는 그걸 보고 최적화를 하구요.그런데 굳이 nullable을 false로 설정해놓고 null 값이 있으면 좋지 않은 결과가 나오겠죠?
