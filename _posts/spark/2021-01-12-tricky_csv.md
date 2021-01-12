---
title: Spark - 까다로운 텍스트 파일 읽기
date: 2021-01-12
tags: Spark
category: programming
sidebar:
    nav: "spark"
---

구분자 두 개와 함께 들어올 수도 있고 안들어올 수도 있는 형식에 맞지 않는 헤더를 가진 텍스트 파일 처리 - DataFrameAPI로는 조금 까다로운 파일을 `textFile` 과 rdd의 `map` 을 이용해서 처리해봅니다.

* 예상 검색 키워드
  * spark read csv with multiple separator
  * scala map split, replace string to spark DataFrame

> Spark 3.0부터는 read 옵션에서 복수개 구분자를 지원합니다. `option("sep", "||")`  
> 따라서 사연이 있어 3.0 버전을 쓸 수 없는 슬픈 사람만 아래를 참고하시면 되겠습니다.  
> [Spark Jira Issue: Support for multiple character delimiter in Spark CSV read](https://issues.apache.org/jira/browse/SPARK-24540)  

따로 발생하면 별 문제가 안되는데 함께 나타나다보니 삽질을 조금 했습니다.

* spark 2.2 버전에서 테스트 하였습니다.

## sample data

아래 데이터를 예제로 합니다.

```scala
sc.textFile("/data/csvtest/info.csv").collect().foreach(println)

// datafile
// col1|^col2|^col,col3
// hi|^hello|^안녕

sc.textFile("/data/csvtest/info2.csv").collect().foreach(println)

// col1|^col2|^col,col3
// hi|^hello|^안녕

```

`|^` 라는 구분자와 함께 `datafile`이라는 의미 없는 헤더가 **간혹** 들어오는 경우를 가정합니다.

```scala
spark.read.csv("/data/csvtest/info.csv").show()
// +---------------+
// |            _c0|
// +---------------+
// |       datafile|
// |col1|^col2|^col| <- 주의 (,col3) 없어짐
// |  hi|^hello|^안녕|
// +---------------+
```

위처럼 DataFrame API로 읽으면 컬럼 하나 날아가는 수가 있습니다. 데이터 안에 `,` 가 있어서 컬럼 하나의 내용이 날아갔습니다.
구분자를 뭘로 하던지 혹시라도 값에 그 구분자가 있으면 원치 않는 상황이 발생합니다.

> 도저히 쓰지 않을 것 같은 글자를 구분자로 지정하는 방법도 있겠지만 *(전각이라던지?)* 이번에는 rdd를 이용해서 처리해보겠습니다.

```scala
var data = sc.textFile("/data/csvtest/info.csv")
val header = data.first
println(s"header: ${header}")

// header 처리
if (!header.contains("|^")){
    println("garbage header comes here!")
    data = data.filter(line=> line != header)
} else {
    println("no header at this time")
}

// multiple separator
case class Info (
    c1: String,
    c2: String,
    c3: String
    )

data
    .map(r=>r.split("\\|\\^")) // 특수문자라서 역슬래쉬 필요
    .map(y=>Info(y(0), y(1), y(2)))
    .toDF() // DataFrame !!
    .show()

// header: datafile
// garbage header comes here!
// +----+-----+--------+
// |  c1|   c2|      c3|
// +----+-----+--------+
// |col1| col2|col,col3|
// |  hi|hello|      안녕|
// +----+-----+--------+
```

이 방법도 값 사이에 `|^` 이 들어가 있다면 구분이 잘못될 수 있습니다. 하지만 안 그러려고 구분자 두 개나 넣어서 만들었을 테니 이 부분은 넘어가도록 합니다.

> 스파크는 csv로 쓰는 경우 값 사이에 구분자가 들어가면 `""`로 구분해줍니다.

```scala
sql("select '컬럼1', '컬럼,,,2', '컬럼3!'").write.csv("/data/csvtest/cc.csv")
// 컬럼1,"컬럼,,,2",컬럼3!
```

끝
