---
title: Install apache zeppelin/spark with Docker
date: 2019-12-28
tags: Spark docker
category: programming
header:
  teaser: /assets/img/post/spark/zeppelin_classic_logo.svg
sidebar:
    nav: "spark"
---

제플린은 주피터와 비슷하게 웹기반의 노트북 스타일 에디터라고 할 수 있습니다. 아파치 재단의 공식 도커 이미지로 제플린을 설치해보겠습니다. 사용할 이미지는 built-in apache spark를 지원합니다. 스파크 사용법을 익히기에 좋습니다.

도커 설치가 미리 되어있어야 합니다. 준비가 되었다면, 이미지를 받아옵니다.

```sh
docker pull apache/zeppelin:0.8.2
# 작성일 기준 최신버전, latest 태그가 없어서 버전 명시가 필요합니다.
```

해당 docker hub에서 dockfile을 읽어보면 이미지가 어떻게 생성되었는지 확인할 수 있습니다. 다양한 의존성들을 설치하고 python, R 관련 라이브러리도 함께 설치되네요.

이미지가 완료되었다면, 아래 명령어로 컨테이너를 띄웁니다.

```sh
docker run -d --rm \
  -p 8080:8080 \
  -v $PWD/logs:/logs \
  -v $PWD/data:/data \
  -v $PWD/notebook:/notebook \
  -e ZEPPELIN_ADDR='0.0.0.0' \
  -e ZEPPELIN_NOTEBOOK_DIR='/notebook' \
  -e ZEPPELIN_LOG_DIR='/logs' \
  --name zeppelin apache/zeppelin:0.8.2
```

위 명령어로 컨테이너를 생성하고 몇 초 ~ 분 후에 `localhost:8080` 경로에서 제플린이 실행되는 것을 확인할 수 있습니다.

물론 현업에서 사용하려면 스파크 세팅 등 많은 환경설정이 필요하겠지만, 스파크와 제플린 사용법을 익히기 위한 용도로는 이정도면 괜찮은 듯 합니다.
