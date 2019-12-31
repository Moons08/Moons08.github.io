---
title: Spark SQL - 로우 분리하기 
date: 2019-02-06
tags: Spark
category: programming
---
데이터를 다루다보면 하나의 로우를 여러개로 분리해야하는 상황이 온다. 다음은 explode 를 활용한 예


### spark < 2.4

```scala
val data = sc.parallelize(Seq(
    """{"userId": 1, "someString": "example1",
        "Date": [20190101, 20190102, 20190103], "val": [1, 2, 9]}""",
    """{"userId": 2, "someString": "example2",
        "Date": [20190101, 20190103, 20190105], "val": [9, null, 6]}"""
))

val df = spark.read.json(data)

df.printSchema
/*
root
 |-- Date: array (nullable = true)
 |    |-- element: long (containsNull = true)
 |-- someString: string (nullable = true)
 |-- userId: long (nullable = true)
 |-- val: array (nullable = true)
 |    |-- element: long (containsNull = true)
*/
```

define `zip` udf:

```scala
import org.apache.spark.sql.functions.{udf, explode}

val zip = udf((xs: Seq[Long], ys: Seq[Long]) => xs.zip(ys))

df.withColumn("result", explode(zip('Date, 'val))).
  select('userId, 'someString,
    $"result._1".alias("date"), $"result._2".alias("value")).
    show
/*
   +------+----------+--------+-----+
   |userId|someString|    date|value|
   +------+----------+--------+-----+
   |     1|  example1|20190101|    1|
   |     1|  example1|20190102|    2|
   |     1|  example1|20190103|    9|
   |     2|  example2|20190101|    9|
   |     2|  example2|20190103| null|
   |     2|  example2|20190105|    6|
   +------+----------+--------+-----+
*/
```

### spark > = 2.4
이미 구현된 `arrays_zip`을 사용하면 간단.

```scala
df.withColumn("result", explode(arrays_zip($"date", $"val"))).
  select($"userId", $"someString", $"result.date", $"result.val").
  show
```

---
