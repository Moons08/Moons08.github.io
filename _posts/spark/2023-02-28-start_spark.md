---
title: 대용량 데이터 처리에 많이 쓰이는 Spark, 여기서 시작하세요
date: 2023-02-28
tags: Spark
category: programming
toc: true
header:
  teaser: /assets/img/post/spark/Apache_Spark_logo.svg
sidebar:
    nav: "spark"
---

Spark를 많이들 쓴다고 해서 어떻게 쓰는지 궁금한데, 어디서 시작해야하는지 모르겠다 싶은 분이 읽으면 좋을 것 같습니다.

Spark는 대규모 데이터 처리를 위한 오픈 소스 분산 처리 프레임워크입니다. Hadoop의 MapReduce와 유사한 분산 처리 엔진을 기반으로 하지만, 메모리 기반 처리와 스트리밍 처리, 머신 러닝, 그래프 처리 등 다양한 기능을 제공합니다. 또한 Spark는 Python, Java, Scala, R 등 다양한 언어를 지원하여 사용자가 편리하게 데이터 처리를 할 수 있도록 합니다. 

1. 프로그래밍이 처음이시라면 먼저 언어에 익숙해지는 것을 추천합니다. python 언어를 공부해보면 어떨까요? 과거 spark는 scala api에 비해 python api가 부족한 부분이 있었는데, 최근에는 이런 부분이 개선되었고, spark에서도 python이 가장 많이 쓰이고 있습니다. 

1. 이미 한가지 이상 언어에 익숙하시다면, [스파크 완벽 가이드](https://www.hanbit.co.kr/store/books/look.php?p_code=B6709029941) 책을 추천합니다. 이 책은 저희 팀에서 데이터 엔지니어로 커리어를 처음 시작하는 분들에게 늘 추천하는 책이고, 데이터 처리에 관심있으신 데이터 분석가  에게도 추천 했었습니다. 핸즈온으로 스파크를 익히기에도 좋고, 스파크가 내부에서는 어떻게 분산처리를 하는지, RDD와 DataFrame은 뭐가 다른지 (spoiler alert; 주로 DataFrame 쓰게 되실 겁니다.) Join hint나 파일 형식에 따라 어떻게 쓰면 좋은지 초심자가 고민할만한 많은 부분을 다루고 있습니다
    
    “스파크 완벽 가이드”는 spark 2 버전 대응이고 지금은 spark 3 버전대가 릴리즈 되었는데요, DataFrame 위주로 처리하는 기조는 크게 바뀌지 않았고 주로 내부 로직이 효율적으로 변하거나 Kubernetes 를 리소스 매니저로 채택하는 등의 변화가 있었습니다. 그래서 이 책으로 지금 공부한다고 해서 나중에 다시 공부해야한다거나 하지는 않습니다.
    
    스파크 완벽 가이드 책을 보면서 핸즈온으로 실습을 하고 싶을 텐데요, [spark 실습 환경 구축 가이드 글](/programming/zeppelin-with-docker) 을 참조하시면 docker로 spark 환경과 대화형 쉘을 바로 로컬 환경에 만들 수 있습니다.

1. 맛보기만 보고 싶다면? 
    
    제가 이전에 쓴 [spark data ETL 가이드](/programming/dataETL)글이 있는데요, spark로 csv, json, parquet 파일을 읽어서 이런저런 transform 처리를 하고, 다시 쓰는 내용이 담겨져 있습니다. 일단 스파크를 사용하시면서 익히고 싶으신 분에게 추천합니다. (scala 로 쓰여 있지만 spark API 사용이 전부라 python 사용법이랑 크게 차이 나지 않아요~)
    
    
