---
title: 도커와 쿠버네티스를 이해하기 위한 로드맵 & 가이드
date: 2021-02-19
tags: docker kubernetes
category: programming
toc: true
--- 

도커, 컴포즈까지는 이제 좀 익숙하게 쓰는데, 쿠버네티스는 뭔가 개념도 어렵고 이해가 잘 안 되더라구요.
좋은 공부 자료를 찾아서 조금 숨통이 트이게 되었는데, 같은 어려움을 가진 분들께 공유하고자 합니다.

도커/쿠버네티스 대한 내용 자체보다는, 이렇게 공부를 했으면 삽질을 좀 덜 했겠다 식의 가이드입니다.

* 추천 독자
  * 도커에 관심은 많은데, 뭐부터 해야 되는지 잘 모르겠는다 싶은 분
  * 쿠버네티스할 거니까 스웜은 넘겨도 된다고 생각하신 분
  * 도커, 컴포즈 예제 잘 따라 해 봤는데 쿠버네티스의 개념이 잘 안 들어오시는 분
  
## 추천 학습 순서

1. 도커
2. 도커 컴포즈
3. 도커 스웜
4. 쿠버네티스

## 도커

도커는 Linux 컨테이너를 만들고 사용할 수 있도록 하는 컨테이너화 기술입니다.

![img](/assets/img/post/docker/docker_run.png)
*커맨드 한 줄에 리눅스 컨네이너가 뙇.*

도커를 왜 써야하는지 모르거나, 컨테이너에 익숙하지 않으시다면 아래 글을 추천합니다.

* [왜 굳이 도커(컨테이너)를 써야 하나요? (44bits)](https://www.44bits.io/ko/post/why-should-i-use-docker-container)
  * 제목처럼 정말 왜 써야 하는지 쓰지 않았을 때의 고통과 씀으로써의 해방을 보여주십니다.
* [초보를 위한 도커 안내서 - 도커란 무엇인가? (subicura)](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)
  * 과거 가상화 방식부터 현재의 컨테이너 기술까지 많은 시각 자료와 함께 다루어주십니다.
* [도커(Docker) 입문편 컨테이너 기초부터 서버 배포까지 (44bits)](https://www.44bits.io/ko/post/easy-deploy-with-docker)
  * 차근차근 도커 핸즈온.

## 도커 컴포즈

도커에 익숙해지고 나면 뭔가 불편함이 느껴집니다. 내가 필요한 이미지 가져다 레이어 덧씌워서 나만의 컨테이너까지 만들었지만,
매번 `docker run -it -어쩌고저쩌고` 옵션 넣어주기가 불편하거든요.

게다가 다루어야 할 컨테이너가 한 개가 아니라 두세 개만 되어도 귀찮음이 엄청납니다. 어플리케이션을 위해서는 하나의 컨네이너로는 안되죠. 모놀리식으로만 서비스를 할 수도 없고요.

이처럼 여러 개의 컨테이너를 다루기 위해 컴포즈를 쓸 수 있습니다.

* postgres db와 django 백엔드 컨테이너를 띄우는 도커 컴포즈 yaml 파일 예시

```yaml
version: '3'

services:
  db:
    image: postgres
    volumes:
      - django_sample_db_dev:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  django:
    build:
      context: .
      dockerfile: ./compose/django/Dockerfile.dev     
    ports:
      - "8000:8000"
    entrypoint: ["sh", "backend/entrypoint.sh"]

volumes:
  django_sample_db_dev: {}  
```

* [도커 컴포즈를 활용하여 완벽한 개발 환경 구성하기 (44bits)](https://www.44bits.io/ko/post/almost-perfect-development-environment-with-docker-and-docker-compose)
  * 정말 완벽한 구성을 위한 글. 도커 컴포즈는 이 글이면 됩니다.
  * django 개발 환경을 구축이 예시로 되어 있습니다.

## 도커 스웜

이제 여러 개의 컨테이너가 여러 대의 서버에서 여러 개의 서비스를 수행합니다. 스웜이나 쿠버네티스는 이를 위한 도구입니다.
클러스터의 규모가 수십대 이내 정도로 작다면 도커 스웜을 추천하더군요. 다만, 스웜이 쿠버네티스에 비해 레퍼런스가 적습니다.

저는 컨네이너 오케스트레이션? 쿠버네티스 공부하면 되는 거 아닌가?라고 생각하고 스웜은 쳐다도 안 봤었습니다.
당연히 쿠버네티스가 상위 호환이라고 생각했거든요. 그런데 그냥 용도가 다른 도구였습니다.
그리고 스웜을 알고 쿠버네티스를 공부하면 정말 이해가 두배 세배는 더 잘됩니다. 스웜이 *그나마* 좀 쉽거든요.

스웜은 도커와 따로 개발되기 시작했지만, 1.12 버전부터는 도커에 편입되었습니다. 그래서 도커만 설치가 되어있으면 스웜 모드(Swarm Mode)를 실행할 수 있습니다.

* [Docker Swarm을 이용한 쉽고 빠른 분산 서버 관리 (subicura)](https://subicura.com/2017/02/25/container-orchestration-with-docker-swarm.html)
  * 17년 2월 글이지만, 오케스트레이션 툴 비교부터 스웜 기능, 핸즈온까지 포함.
  * 이미지, 실습 구동 영상도 있어서 이해하기 좋아요.
* [[Infra] Kubernetes vs Swarm vs Mesos 비교](https://chrisjune-13837.medium.com/infra-kubernetes-vs-swarm-vs-mesos-%EB%B9%84%EA%B5%90-b04b2cd032ab)
  * 17년 기준) 중소형 클러스터-> 스웜, 중대형 -> 쿠버, 초대형 -> 메소스
  * 18년 이후 부터는 쿠버네티스로 대동단결.
  * [원문 링크](https://www.loomsystems.com/blog/single-post/2017/06/19/kubernetes-vs-docker-swarm-vs-apache-mesos-container-orchestration-comparison)

## 쿠버네티스

도커(엔진), 컴포즈, 스웜까지 익숙해지셨다면 쿠버네티스를 마음껏 공부하시면 되겠습니다. ~~저도 이제 더 하려구요.~~

* [쿠버네티스 시작하기 - Kubernetes란 무엇인가? (subicura)](https://subicura.com/2019/05/19/kubernetes-basic-1.html)
* [쿠버네티스 안내서 (subicura)](https://subicura.com/k8s/)

## 추천 도서

* [시작하세요! 도커/쿠버네티스](https://wikibook.co.kr/docker-kube/)
  * 굉장히 좋습니다. 600 페이지 중 반은 도커(엔진+스웜+컴포즈), 반은 쿠버네티스로 구성되어 있습니다. 도커 엔진 부분 분량이 많아요.
  * 1장의 도커 엔진 보다가 좀 어렵다 싶으시면 뒷부분까지 쭉쭉 읽으시고 다시 정독하시는 걸 추천합니다.
  * 아쉽게도 전자책 출판 계획이 당분간(2021년 1월 기준)은 없답니다...

* 매니징 쿠버네티스
  * 쿠버네티스 공동 창시자가 썼다는데, 저한테는 너무 어려웠습니다. 위의 책 마저 읽고 다시 읽어봐야겠습니다.
