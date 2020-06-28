---
title: Hive Partition 다루기
date: 2020-06-27
tags: Spark
category: programming
toc: True
header:
  teaser: 
sidebar:
    nav: "spark"
---

파티셔닝으로 데이터를 분할함으로써 쿼리가 스캔하는 데이터의 양을 제한하여 성능을 향상시킬 수 있습니다. 관리도 훨씬 편해집니다.

## 파티션 테이블 생성

```sql
CREATE EXTERNAL TABLE tb_sample (
    userid BIGINT,
    viewTime INT
)
PARTITIONED BY (year int, month int, day int) -- 다중 레벨 파티션
-- PARTITIONED BY (dt STRING) -- 단일 파티션
STORED AS PARQUET
LOCATION '/data/tb_sample'
```

다중 레벨 파티션을 설정하면 hdfs나 s3에 계층적으로 파일이 저장됩니다. 예를 들면 아래와 같습니다.

```sh
root@2b675156384c:/# hadoop fs -ls -R / | grep "^d" |grep tb_sample
drwxr-xr-x - root supergroup 0 2020-06-28 04:43 /data/tb_sample
drwxr-xr-x - root supergroup 0 2020-06-28 04:43 /data/tb_sample/year=2020
drwxr-xr-x - root supergroup 0 2020-06-28 04:43 /data/tb_sample/year=2020/month=6
drwxr-xr-x - root supergroup 0 2020-06-28 04:43 /data/tb_sample/year=2020/month=6/day=23
drwxr-xr-x - root supergroup 0 2020-06-28 04:43 /data/tb_sample/year=2020/month=6/day=24
drwxr-xr-x - root supergroup 0 2020-06-28 04:43 /data/tb_sample/year=2020/month=6/day=25
```

## 파티션 조작

### 파티션 추가

관리형(Internal) 테이블을 사용할때는 insert 후에 자동으로 파티션이 추가됩니다.

```sql
ALTER TABLE tb_sample ADD PARTITION (year=2020, month=06, day=23)
LOCATION '/data/tb_sample/year=2020/month=6/day=23';
```

### 파티션 삭제

하나의 파티션 혹은 범위로 파티션을 삭제할 수 있습니다. 외부 테이블인 경우 스키마만 사라질 뿐, 해당 경로의 데이터는 사라지지 않습니다.

```sql
-- 2020년 아래 파티션 모두 삭제
ALTER TABLE tb_sample DROP PARTITION(year=2020);

-- 2020년 6월 파티션 삭제
ALTER TABLE tb_sample DROP PARTITION(year=2020, month=06);

-- 2020년 6월 23일 이후 파티션 삭제
ALTER TABLE tb_sample DROP PARTITION(year=2020, month=06, day>23);
+-------------------+---------------------+-----------------+------------------+----------------+
| tb_sample.userid  | tb_sample.viewtime  | tb_sample.year  | tb_sample.month  | tb_sample.day  |
+-------------------+---------------------+-----------------+------------------+----------------+
| 4214              | 32                  | 2020            | 6                | 23             |
+-------------------+---------------------+-----------------+------------------+----------------+

-- 2020년 6월 24일 이전이거나 25일 이후 파티션 삭제
ALTER TABLE tb_sample DROP PARTITION(day<'24'), PARTITION(day>'25'); -- and 아니고 or 조건으로 적용됨
+-------------------+---------------------+-----------------+------------------+----------------+
| tb_sample.userid  | tb_sample.viewtime  | tb_sample.year  | tb_sample.month  | tb_sample.day  |
+-------------------+---------------------+-----------------+------------------+----------------+
| 4214              | 33                  | 2020            | 6                | 24             |
| 4214              | 13                  | 2020            | 6                | 25             |
+-------------------+---------------------+-----------------+------------------+----------------+
```

### 파티션 복구

```sql
MSCK REPAIR TABLE tb_sample
```

위 명령으로 외부 테이블의 파티션을 **한꺼번에** 추가할 수 있습니다.
Hive와 호환되는 형식으로 저장된 경우에만 적용됩니다. 호환되지 않는 경우에는 `ALTER TABLE ADD PARITION`를 사용해야 합니다. *MSCK는 metastore check의 약자입니다.*  

### 파티션 조회

```sql
SHOW PARTITION tb_sample;
```

### 파티션 이름 변경

```sql
alter table tb_2 partition (dt='20200623') rename to partition (dt='20210623')
```

## 동적 파티션

해당 파티션이 없는 경우에 파티션을 만들어서! 입력해주는 방법입니다.

```sql
FROM page_view_stg pvs
INSERT OVERWRITE TABLE page_view PARTITION(dt='2008-06-08', country='US')
       SELECT pvs.viewTime, null, pvs.ip WHERE pvs.country = 'US'
INSERT OVERWRITE TABLE page_view PARTITION(dt='2008-06-08', country='CA')
       SELECT pvs.viewTime, null, pvs.ip WHERE pvs.country = 'CA'
INSERT OVERWRITE TABLE page_view PARTITION(dt='2008-06-08', country='UK')
       SELECT pvs.viewTime, null, pvs.ip WHERE pvs.country = 'UK'
...
...
...
INSERT OVERWRITE TABLE page_view PARTITION(dt='2008-06-08', country='KO')
       SELECT pvs.viewTime, null, pvs.ip WHERE pvs.country = 'KO';
```

동적 파티션을 사용한다면 위 쿼리를 아래처럼 바꿀 수 있습니다.

```sql
FROM page_view_stg pvs
INSERT OVERWRITE TABLE page_view PARTITION(dt='2008-06-08', country)
       SELECT pvs.viewTime, null, pvs.ip, pvs.country
```

편리할 뿐만아니라, 인서트 문이 하나라서 맵리듀스 작업도 하나가 되어 성능 또한 향상된답니다.
