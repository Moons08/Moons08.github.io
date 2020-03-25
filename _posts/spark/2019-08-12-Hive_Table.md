---
title: Hive table 다루기
date: 2019-08-12
tags: SQL Hadoop
category: programming
toc: True
sidebar:
    nav: "spark"
---

하둡&제플린 환경에서 유저에게 데이터를 사용할 수 있도록 hive 테이블을 만들어두면 편리합니다.

## Create Hive Table

### internal vs external table

하이브 테이블 생성은 여타 db와 비슷합니다. 다만, 저장될 형식이라던지, location 등을 설정하는 방식이 조금 다릅니다. internal table은 일반 db를 생각하면 됩니다. external table은 이미 하둡에 데이터가 있는 데이터를 기반으로 테이블을 만듭니다. 스키마만 정해주면 됩니다. 그래서 파일 따로, 스키마 따로 관리하기 좋습니다.

### with csv

가장 많이 쓰이는 포맷인 것 같습니다.

```sql
CREATE EXTERNAL TABLE table_name(
    col1 STRING,
    col2 INT
    )
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/file/path/some/where';
```

### with parquet

parquet는 컬럼 기반 저장 포맷으로, 스키마와 함께 데이터를 압축하여 저장합니다.
데이터를 작고 효율적으로 쓸 수 있습니다. 압축형태라 Hue에서 내용을 확인할 수 없다는게 단점..

```sql
CREATE EXTERNAL TABLE table_name(
    col1 STRING,
    col2 INT
    )
STORED AS PARQUET
LOCATION '/file/path/some/where';
```

### with json

유연성이 높은 json입니다. 쌓아야할 데이터 형태가 바뀔 가능성이 있다면 json이죠.

```sql
CREATE EXTERNAL TABLE table_name(
    col1 STRING,
    col2 STRING,
    col3 STRUCT <
        s_1:STRING,
        s_2:INT
    >
)ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
STORED AS TEXTFILE
```

## Partition

파티션을 나눠주면 여러모로 관리가 편합니다.

- 재작업시 해당 파티션 밀어버리고 재생성(`overwrite`)
- where 조건에 포함하여 쿼리 시간 단축

```sql
CREATE EXTERNAL TABLE table_name(
    col1 STRING,
    col2 INT
    )
PARTITIONED BY (year INT, month STRING)
STORED AS PARQUET
LOCATION '/file/path/some/where';

ALTER TABLE table_name ADD 
IF NOT EXIST PARTITION (year=2019, month='08');
```

## Table Options

단순히 테이블 만들고, 파티션 정도 나눠주면 행복할 것 같지만, 새로운 문제는 언제든 나오기 마련입니다.

### 파일 인코딩 변경

한글 데이터의 경우, `euc-kr` 등 해외에서는 잘 안쓰는 인코딩으로 되어있기 때문에,
  변경이 필요할 때가 있습니다. (기본값은 `utf-8`)

```sql
ALTER TABLE table_name SET SERDEPROPERTIES ('serialization.encoding'='euc-kr');
```

## 마무리

짬나는대로 자주 사용하는 hive 관련 옵션에 대해 추가할 예정입니다.
