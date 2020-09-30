---
title: Spark with JDBC
date: 2020-09-20
tags: Spark
category: programming
toc: False
header:
  teaser: /assets/img/post/spark/Apache_Spark_logo.svg
sidebar:
    nav: "spark"
---

jdbc driver를 이용해서 spark로 db 데이터를 읽거나 쓸 수 있습니다.

사용 환경설정(spark-shell, 제플린 인터프리터 등)이 필요합니다. 아래처럼 스파크 클래스패스에 jdbc 드라이버를 추가하고 jar 파일을 제공해야 합니다.  
`./bin/spark-shell --driver-class-path postgresql-9.4.1207.jar --jars postgresql-9.4.1207.jar`

## 읽기

```python
# python
# 방법 1
jdbcDF = spark.read \
    .format("jdbc") \
    .option("url", "jdbc:postgresql:dbserver") \
    .option("dbtable", "schema.tablename") \
    .option("user", "username") \
    .option("password", "password") \
    .load()

# 방법 2
url = "jdbc:postgresql:dbserver"
props = {"user": "username", "password": "password"}

jdbcDF2 = spark.read \
    .jdbc(url, "schema.tablename", properties=props)
```

```scala
// scala
val url = "jdbc:postgresql:dbserver"
val props = new Properties()
props.put("driver", "org.postgresql.Driver")
        // url 접속시 사용할 드라이버 클래스명
        // cf) oracle.jdbc.driver.OracleDriver
props.put("user", "username")
props.put("password", "password")

// Specifying the custom data types of the read schema
props.put("customSchema", "id DECIMAL(38, 0), name STRING")
val jdbcDF3 = spark.read
  .option("numPartitions", 10) //  maximum number of concurrent JDBC connections.
  .jdbc(url, "schema.tablename", props)
```

예시를 spark 3.0.1 문서에서 가져왔는데, 읽기 옵션에 커스텀 스키마를 적용할 수도 있네요.  
~~왜 예제에는 props에 driver 넣는게 없지?~~

## 쓰기

```python
# python
jdbcDF \
.write \
.jdbc(url, "schema.tablename", mode="overwrite", properties=props)
```

```scala
// scala
spark.sql("SELECT ID, VALUE FROM sparkTemp")
  .write.mode("overwrite")
  .option("truncate", true)  // default: false
//   .option("createTableColumnTypes", "ID CHAR(64), VALUE VARCHAR(1024)")
  .jdbc(url, "schema.tablename", props)
```

### 주의

* 오라클 db에 데이터를 쓸 때, 컬럼명을 소문자로 쓰면 `id -> "id"` 이런 식으로 따옴표가 같이 들어가는 경우가 있습니다. 그래서 저는 꼭 **대문자**로 씁니다. (관련하여 이유를 아시는 분이 있다면 말씀해주시면 감사하겠습니다.)
* ~~`truncate` 옵션 **없이** 덮어쓰기를 할 경우에는 인덱스, 제약조건, 코멘트 등 대상 테이블에 딸린 다양한 것들이 사라집니다. (무결성 깨지는 경우가 생김)~~
  * truncate 해도 날아가는 경우가 있음.. 확인 필요
