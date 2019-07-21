---
title: \[Airflow] docker를 활용한 초간단 설치하기
date: 2019-07-20
tags: airflow docker
category: programming
---
airflow docker 설치  
docker를 이용하여 airflow를 로컬에 설치하던 것보다 더 쉽게 설치해보겠습니다.  
에어플로우를 더 아름답게 쓰기 위해서는 executor, db 설정이 필요한데, 
모든 환경설정이 그렇듯이 설치할 부품들이 늘어날수록 고통도 늘어납니다. 
이런 상황에서 docker는 그런 고통들을 줄여주는 아주 좋은 도구입니다.  


## 설치
  <!--도커 설치를 위해서는 [여기]()를 참조해 주세요.-->  
dockfile과 이미지를 잘 만들어놓은 레포를 이용하겠습니다.

```bash
$ git clone https://github.com/puckel/docker-airflow
$ cd docker-airflow
$ tree
.
├── Dockerfile
├── LICENSE
├── README.md
├── config
│   └── airflow.cfg
├── dags
│   ├── __pycache__
│   │   ├── test.cpython-36.pyc
│   │   └── tuto.cpython-36.pyc
│   └── tuto.py
├── docker-compose-CeleryExecutor.yml
├── docker-compose-LocalExecutor.yml
└── script
    └── entrypoint.sh
```
  
  
그리고 도커 이미지를 도커허브에서 받아옵니다. 이건 시간이 조금 걸려요
```bash
docker pull puckel/docker-airflow 
```

놀랍게도 준비가 끝났습니다. 이제 실행만하면 됩니다.

```bash
$ docker-compose -f docker-compose-LocalExecutor.yml up -d 
$ docker ps

CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS                       PORTS                                        NAMES
968e2056471a        puckel/docker-airflow:1.10.3   "/entrypoint.sh webs…"   About an hour ago   Up About an hour (healthy)   5555/tcp, 8793/tcp, 0.0.0.0:8080->8080/tcp   airflow_webserver_1
1d2dfb29efe4        postgres:9.6                   "docker-entrypoint.s…"   About an hour ago   Up About an hour             5432/tcp                                     airflow_postgres_1
```

기본 세팅으로 docker container의 dags를 로컬의 dags폴더에 마운트하게 됩니다.
여기에서 이제 airflow 사용만 해보겠다! 하시면 dag 폴더에 들어가셔서 다양한 task들을 시도해보시면 됩니다. 
로컬에서 설치했던 것과 같이 http://localhost:8080으로 들어가면 웹서버를 볼 수 있습니다.  
  
아래에서는 방금 실행한 yaml 파일과 함께 환경 구성이 어떻게 되어있나 확인해 보겠습니다.  

## 구성환경 뜯어보기

구성환경을 확인하기 위해 docker-compose로 실행하는 yaml, docker 이미지를 생성할 때 쓴 Dockfile, 
그리고 컨테이너가 올라올 때 실행되는 ENTRYPOINT까지 세개의 파일을 보겠습니다.

먼저 `docker-compose-LocalExecutor.yml` 파일을 보겠습니다. 

```yaml
version: '2.1' 
        # docker-compose 버전입니다. 작성일 기준으로 3.0을 권장하지만 아직 2.1을 쓰고 있네요.  
        # 서비스는 `postgres`와 `webserver` 두개로 구성되어 있습니다. 

services:
    postgres:
        image: postgres:9.6                 # db는 postgres 공식 이미지를 가져옵니다.
        environment:                        # db 구성에 필요한 값들을 넣어줍니다.
            - POSTGRES_USER=airflow
            - POSTGRES_PASSWORD=airflow
            - POSTGRES_DB=airflow

    webserver:                              # 여기서 부터 airflow 네요.
        image: puckel/docker-airflow:1.10.3 # puckel 계정 아래 있는 이미지 사용
        restart: always                     # 컨테이너 중단된 경우, 자동으로 재시작 해주는 옵션
        depends_on:
            - postgres                      # postgres를 db로 사용하니까 의존성 설정을 해줍니다.
        environment:
            - LOAD_EX=n                     # 여기는 이미지를 구성할때 설정한 환경변수를 넣어줍니다.
            - EXECUTOR=Local                # 아래에서 이미지 구성 파일을 보며 확인하겠습니다.
        volumes:
            - ./dags:/usr/local/airflow/dags # 현재 경로의 dags 폴더를 컨테이너 dags에 마운트합니다.
            # Uncomment to include custom plugins
            # - ./plugins:/usr/local/airflow/plugins
        ports:
            - "8080:8080"                   # 컨테이너 port 8080을 localhost의 8080으로 맞춰줍니다.
        command: webserver                  # 다 끝나면 웹서버 명령어로 ui를 띄우네요.
        healthcheck:                        # -f 로 webserver pid가 잘 생성이 되었는지 확인합니다.
            test: ["CMD-SHELL", "[ -f /usr/local/airflow/airflow-webserver.pid ]"]
            interval: 30s  # 30초 마다 하네요.
            timeout: 30s   # 30초 동안 기다려 주고
            retries: 3     # 3번 재시도.. 합니다. 자주 죽어서 쓰나봅니다.
```

webserver는 띄우는데 scheduler 띄우는 부분이 없습니다. test task들은 잘 동작하는데 말이죠.  
`Dockfile`을 보면 알 수 있겠죠.  

```bash
# VERSION 1.10.3
# AUTHOR: Matthieu "Puckel_" Roisil
# DESCRIPTION: Basic Airflow container
# BUILD: docker build --rm -t puckel/docker-airflow .
# SOURCE: https://github.com/puckel/docker-airflow

... 

COPY script/entrypoint.sh /entrypoint.sh
COPY config/airflow.cfg ${AIRFLOW_USER_HOME}/airflow.cfg

RUN chown -R airflow: ${AIRFLOW_USER_HOME}

EXPOSE 8080 5555 8793 

USER airflow
WORKDIR ${AIRFLOW_USER_HOME}
ENTRYPOINT ["/entrypoint.sh"] 
CMD ["webserver"] # set default arg for entrypoint
```
이런저런 환경설정 및 설치부분을 지나서 아랫부분에 뭔가 있네요.
그런데 여기서도 scheduler 실행부분은 없습니다. 컨테이너가 올라갈때 실행되는 `entrypoint.sh`을 보면 나올 것 같습니다.


`script/entrypoint.sh`
```bash
...

# Defaults and back-compat
: "${AIRFLOW__CORE__FERNET_KEY:=${FERNET_KEY:=$(python -c "from cryptography.fernet import Fernet; FERNET_KEY = Fernet.generate_key().decode(); print(FERNET_KEY)")}}"
: "${AIRFLOW__CORE__EXECUTOR:=${EXECUTOR:-Sequential}Executor}"

...

case "$1" in
  webserver)
    airflow initdb
    if [ "$AIRFLOW__CORE__EXECUTOR" = "LocalExecutor" ]; then
      # With the "Local" executor it should all run in one container.
      airflow scheduler &
    fi
    exec airflow webserver
```
여기네요, webserver 명령어가 들어온 경우 일단 initdb를 날리고,  
거기에 EXECUTOR를 Local로 설정한 경우에는 webserver와 scheduler를 한 컨테이너에서 실행시킵니다.


에어플로우 설치를 공부하면서 도커와 쉘 스크립트 공부까지하게 되네요. Celery를 사용한 구성환경은 다음 포스트에서 다루도록 하겠습니다.
