---
title: Spark Bucketed Table의 Join Strategy와 Config
date: 2023-05-15
tags: Spark
category: programming
toc: true
header:
  teaser: /assets/img/post/spark/Apache_Spark_logo.svg
sidebar:
    nav: "spark"
---

Spark bucketing의 특징과 잘 쓰기 위한 방법에 대해 정리합니다.

### Bucketed Table's Aggregation & pruning

Bucketed Table을 만들 때는 Aggregation이나 Filter에 주요하게 사용될 컬럼을 Bucket Key로 설정하는 것이 중요합니다. 

```
tableA.groupBy('user_id').agg(count('*'))  
# 이 코드는 부분적으로 셔플이 발생합니다.

tableA.withColumn('n', count('*').over(Window().partitionBy('user_id')))
# 이 코드는 전체 셔플이 발생합니다.
```

그런데 만약 tableA가 user_id로 버케팅 되어있다면 셔플이 전혀 발생하지 않습니다.

```
spark.table('tableA').filter(col('user_id') == 123)
# 이 코드는 tableA 의 모든 data file을 읽습니다. 
```
만약 버케팅 되어 있다면 저 레코드가 위치한 파일만 읽습니다. 

또 언제 셔플이 발생할까요? 다른 테이블과 Join할 때 발생합니다. 아래부터는 Bucketed Table의 Join 관련 내용을 정리합니다.

### Number of Shuffle Partition <= Number Of Bucket

셔플 파티션 수가 버켓의 수보다 크면 Bucketed Table에 셔플이 발생합니다. 
따라서 Bucketed Table을 조인에 사용할 때는 셔플 파티션 수(`spark.sql.shuffle.partitions`)가 Bucket의 수보다 같거나 작아야 합니다. 

### Number of Bucket이 다른 두 개의 Bucketed Table을 조인

숫자가 큰 테이블이 숫자가 작은 테이블과 같아지도록 파티션이 합쳐집니다. 만약 50개로 Bucketing 된 A 테이블과 100개로 Bucketing 된 B 테이블을 조인할 때 B 테이블이 셔플되지 않고 50개의 Bucket 으로 병합(coalesce)이 됩니다. 두 테이블 모두 셔플 파티션 수보다 버켓 수가 작다면 둘 다 셔플이 발생합니다.
3.1.1 부터 지원되는 `spark.sql.bucketing.coalesceBucketsInJoin.enabled` 옵션이 켜져 있어야합니다. 아니라면 한쪽 테이블에는 항상 셔플이 발생합니다. 

### Sort

원한다면 불필요한 정렬도 없앨 수 있습니다. 다만, 아래 조건을 충족해야합니다.

1. 버킷당 파일이 하나만 있어야합니다.
  * Spark의 Bucketing은 Bucket 당 파일이 여러개 생길 수 있고, 하나의 파일 안에서는 순서가 보장되지만 여러 개의 파일 중에서는 순서가 보장되지 않습니다. 
2.	`spark.sql.legacy.bucketedTableScan.outputOrdering` 옵션이 True 거나 Spark version이 3.0 미만 이어야 합니다. 
  * Spark 3.0 이전에는 Bucket 당 파일이 하나인지 확인하는 과정이 있었고 비효율적이라 3.0 이후 부터는 기본적으로 모두 정렬하게 해버렸습니다.


### 유의사항

* `spark.sql.sources.bucketing.enabled` default는 True인데, 만약 False라면 bucketed table을 그냥 normal table로 취급합니다.
* `spark.sql.sources.bucketing.autoBucketedScan.enabled` 이것도 default는 True인데, Bucketing 이점을 누리지 못하는 상황에는 normal table로 취급하는 옵션입니다.
* Bucketed Table을 쿼리하는 경우, 첫번째 stage에서는  Bucket의 수만큼 Task가 생깁니다. 시간이 지나면 하나의 버켓 사이즈가 커질수 있는데요, 그런 경우에는 partition 사이즈도 커질 것이고, data spill이 발생할 수도 있습니다. 따라서 버켓 테이블을 생성할 때 이부분을 고려해야합니다. Disk spill 이 발생하면 Bucketed 옵션을 끄고 partition 수를 늘려서 일반 테이블처럼 다루는게 나을 수 있습니다. 

---

참고

https://towardsdatascience.com/best-practices-for-bucketing-in-spark-sql-ea9f23f7dd53
