---
title: \[Airflow] docker를 활용한 초간단 설치하기 2
date: 2019-07-21
tags: airflow docker
category: programming
---

지난 포스트에 이어 CELERY EXECUTOR를 사용하는 에어플로우, 도커로 설치하기 입니다.  
설치 앞부분을 위한 깃레포, 도커이미지를 받아오는 부분은
 [이전 포스트](https://moons08.github.io/programming/airflow-with-docker/)를 확인해주세요.


### CeleryExecutor 

```bash
$ docker stop $(docker ps -aq) # 도커로 뭔가 실행중이라면 일단 멈춰주고,
$ docker-compose -f docker-compose-CeleryExecutor.yml up -d # 이번에는 이 yaml 파일로 띄워봅니다.
$ docker ps

CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS                            PORTS                                        NAMES
7e2a4556a97a        puckel/docker-airflow:1.10.3   "/entrypoint.sh work…"   5 seconds ago       Up 3 seconds                      5555/tcp, 8080/tcp, 8793/tcp                 airflow_worker_1
b8264ff5a2a5        puckel/docker-airflow:1.10.3   "/entrypoint.sh sche…"   6 seconds ago       Up 5 seconds                      5555/tcp, 8080/tcp, 8793/tcp                 airflow_scheduler_1
b42f37b83f69        puckel/docker-airflow:1.10.3   "/entrypoint.sh webs…"   9 seconds ago       Up 6 seconds (health: starting)   5555/tcp, 8793/tcp, 0.0.0.0:8080->8080/tcp   airflow_webserver_1
359b67ec29ae        puckel/docker-airflow:1.10.3   "/entrypoint.sh flow…"   9 seconds ago       Up 6 seconds                      8080/tcp, 0.0.0.0:5555->5555/tcp, 8793/tcp   airflow_flower_1
69c1ece9dc38        postgres:9.6                   "docker-entrypoint.s…"   10 seconds ago      Up 8 seconds                      5432/tcp                                     airflow_postgres_1
80bacd716140        redis:3.2.7                    "docker-entrypoint.s…"   10 seconds ago      Up 8 seconds                      6379/tcp                                     airflow_redis_1
```
CeleryExecutor를 사용하는 yml 파일을 실행할 경우 위와 같습니다.
LocalExecutor 실행했던 것과 다르게 컨테이너들이 뭔가 많이 생겼습니다. 이름을 보니 postgres 이건 db고, webserver, scheduler.. 
익숙한 것들도 있네요. 실습을 위한 환경구성은 위의 명령어 하나로 끝입니다. 조금 더 자세히 보실 분을 계속 읽어주세요.

### yaml file 

위에서 `docker-compose`로 실행했던 `docker-compose-CeleryExecutor.yml`파일을 찬찬히 읽어 보겠습니다.
 서비스는 6개, webserver, scheduler(airflow), postgres(DB), 
 redis(MQ), celery(worker), flower(worker monitor)로 구성 되어있네요. 

```yaml
version: '2.1'
services:
    redis:
        image: 'redis:3.2.7' # 먼저 mq를 담당할 redis입니다.
        command: redis-server --requirepass redispass
                  # 이전에 rabbitMQ랑 db 연동해서 설치한다고 고생 좀 했었는데, 도커로 하니 좋네요.
                  # `requirepass` 부분으로 redis 비밀번호를 설정합니다.
    postgres:
        image: postgres:9.6 # 여기서도 db로 postgres를 씁니다. 
        environment:
            - POSTGRES_USER=airflow
            - POSTGRES_PASSWORD=airflow # db 로그인 설정을 해주고
            - POSTGRES_DB=airflow
            - PGDATA=/var/lib/postgresql/data/pgdata # db의 경로를 설정해 줍니다.
        volumes:
            - ./pgdata:/var/lib/postgresql/data/pgdata # 위에서 설정한 db 경로를 마운트 해줍니다.

    webserver:
        image: puckel/docker-airflow:1.10.3
        restart: always 
        depends_on:
            - postgres # db
            - redis    # mq
        environment:
            - LOAD_EX=n
            - FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho= 
                    # endpoint.sh에서 본 것 같아요.
                    # airflow.cfg를 보니 이렇게 설명해주네요. 
                    # 'Secret key to save connection pass in the db' 
                    
            - EXECUTOR=Celery # executor를 설정 해줍니다.
            
            - POSTGRES_USER=airflow
            - POSTGRES_PASSWORD=airflow
            - POSTGRES_DB=airflow
            - REDIS_PASSWORD=redispass 
                    # 깃헙에서 받은 파일에서는 db 연동, redis 연동 부분이 주석처리 되어있는데, 
                    # entrypoint.sh에 위의 파라미터들이 기본값으로 입력되어 있습니다.
                    # 실 환경에서 사용할 때는 마찬가지로 바꿔주어야 합니다.
        volumes:
            - ./dags:/usr/local/airflow/dags
            # Uncomment to include custom plugins
            # - ./plugins:/usr/local/airflow/plugins
        ports:
            - "8080:8080"
        command: webserver
        healthcheck:
            test: ["CMD-SHELL", "[ -f /usr/local/airflow/airflow-webserver.pid ]"]
            interval: 60s
            timeout: 30s
            retries: 3
```

FERNET_KEY는 db 안에서의 암호화된 통신을 위한 키였습니다. LocalExecutor 실행시에는 하나의 컨테이너에서 해서 굳이 설정할 필요가 없었던거겠죠?
이번에는 서로 다른 컨테이너(webserver, scheduler, worker)에서 db에 연결해야하는 상황이라서 키를 통일시켜주어야 했습니다. 실 환경에서는 다음 명령어로 키를 생성해서 바꿔주세요.  
`docker run puckel/docker-airflow python -c "from cryptography.fernet import Fernet; FERNET_KEY = Fernet.generate_key().decode(); print(FERNET_KEY)"`


```yaml
    flower: # flower는 샐러리 워커들의 상황을 모니터링하는 web ui입니다. 
        image: puckel/docker-airflow:1.10.3
        restart: always
        depends_on:
            - redis
        environment:
            - EXECUTOR=Celery
            - REDIS_PASSWORD=redispass
        ports:
            - "5555:5555"
        command: flower

    scheduler:
        image: puckel/docker-airflow:1.10.3
        restart: always
        depends_on:
            - webserver # scheduler를 db, mq에 의존성 설정을 할 줄 알았는데 webserver만 보고 있네요.
                        # 처음에 띄울때 설정만 따라가나보다, depends_on의 depends_on 느낌으로.
        volumes:
            - ./dags:/usr/local/airflow/dags
            # Uncomment to include custom plugins
            # - ./plugins:/usr/local/airflow/plugins
        environment:
            - LOAD_EX=n
            - FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho=
            - EXECUTOR=Celery
            - POSTGRES_USER=airflow
            - POSTGRES_PASSWORD=airflow
            - POSTGRES_DB=airflow
            - REDIS_PASSWORD=redispass
        command: scheduler

    worker: # 실제 작업을 수행하는 worker를 띄우는 컨테이너입니다. 
        image: puckel/docker-airflow:1.10.3
        restart: always
        depends_on:
            - scheduler
        volumes:
            - ./dags:/usr/local/airflow/dags
            # Uncomment to include custom plugins
            # - ./plugins:/usr/local/airflow/plugins
        environment:
            - FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho=
            - EXECUTOR=Celery
            - POSTGRES_USER=airflow
            - POSTGRES_PASSWORD=airflow
            - POSTGRES_DB=airflow
            - REDIS_PASSWORD=redispass
        command: worker 
```

각각의 컨테이너들은 띄워진 이후에 커맨드를 실행합니다. 
`./script/entrypoint.sh` 를 보면 이후의 명령어들을 명확하게 볼 수 있습니다. 

```bash
#!/usr/bin/env bash

TRY_LOOP="20"

: "${REDIS_HOST:="redis"}"
: "${REDIS_PORT:="6379"}"
: "${REDIS_PASSWORD:=""}"

: "${POSTGRES_HOST:="postgres"}"
: "${POSTGRES_PORT:="5432"}"
: "${POSTGRES_USER:="airflow"}"
: "${POSTGRES_PASSWORD:="airflow"}"
: "${POSTGRES_DB:="airflow"}"  # 디폴트 계정정보를 설정해놓은 부분

# Defaults and back-compat
: "${AIRFLOW__CORE__FERNET_KEY:=${FERNET_KEY:=$(python -c "from cryptography.fernet import Fernet; FERNET_KEY = Fernet.generate_key().decode(); print(FERNET_KEY)")}}"
: "${AIRFLOW__CORE__EXECUTOR:=${EXECUTOR:-Sequential}Executor}"
            # 위에서 본 FERNEY_KEY와 executor 설정 변수를 담는 부분 

export \
  AIRFLOW__CELERY__BROKER_URL \
  AIRFLOW__CELERY__RESULT_BACKEND \
  AIRFLOW__CORE__EXECUTOR \
  AIRFLOW__CORE__FERNET_KEY \
  AIRFLOW__CORE__LOAD_EXAMPLES \
  AIRFLOW__CORE__SQL_ALCHEMY_CONN \


# Load DAGs exemples (default: Yes)
if [[ -z "$AIRFLOW__CORE__LOAD_EXAMPLES" && "${LOAD_EX:=n}" == n ]]
then
  AIRFLOW__CORE__LOAD_EXAMPLES=False
fi

# Install custom python package if requirements.txt is present
if [ -e "/requirements.txt" ]; then
    $(which pip) install --user -r /requirements.txt
fi

if [ -n "$REDIS_PASSWORD" ]; then
    REDIS_PREFIX=:${REDIS_PASSWORD}@
else
    REDIS_PREFIX=
fi
```

아래는 컨테이너들이 올라오기 전에 다음 작업을 하려고하면 에러가 나는 것을 방지하기 위한 부분이네요.

```bash
wait_for_port() {
  local name="$1" host="$2" port="$3"
  local j=0
  while ! nc -z "$host" "$port" >/dev/null 2>&1 < /dev/null; do
    j=$((j+1))
    if [ $j -ge $TRY_LOOP ]; then
      echo >&2 "$(date) - $host:$port still not reachable, giving up"
      exit 1
    fi
    echo "$(date) - waiting for $name... $j/$TRY_LOOP"
    sleep 5
  done
} 
```

아래는 우리가 설정한 익스큐터 설정에 따른 airflow config 변수들을 바꾸는 부분입니다.

```bash
if [ "$AIRFLOW__CORE__EXECUTOR" != "SequentialExecutor" ]; then
  AIRFLOW__CORE__SQL_ALCHEMY_CONN="postgresql+psycopg2://$POSTGRES_USER:$POSTGRES_PASSWORD@$POSTGRES_HOST:$POSTGRES_PORT/$POSTGRES_DB"
  AIRFLOW__CELERY__RESULT_BACKEND="db+postgresql://$POSTGRES_USER:$POSTGRES_PASSWORD@$POSTGRES_HOST:$POSTGRES_PORT/$POSTGRES_DB"
  wait_for_port "Postgres" "$POSTGRES_HOST" "$POSTGRES_PORT"
fi

if [ "$AIRFLOW__CORE__EXECUTOR" = "CeleryExecutor" ]; then
  AIRFLOW__CELERY__BROKER_URL="redis://$REDIS_PREFIX$REDIS_HOST:$REDIS_PORT/1"
  wait_for_port "Redis" "$REDIS_HOST" "$REDIS_PORT"
fi
```

마지막으로 우리가 찾던 컨테이너가 올라온 뒤에 실행하는 명령어를 날려주는 부분입니다.

```bash
case "$1" in
  webserver)
    airflow initdb
    if [ "$AIRFLOW__CORE__EXECUTOR" = "LocalExecutor" ]; then
      # With the "Local" executor it should all run in one container.
      airflow scheduler & # 이번엔 샐러리익스큐터니까 따로 실행되겠습니다.
    fi
    exec airflow webserver
    ;;
  worker|scheduler)
    # To give the webserver time to run initdb.
    sleep 10
    exec airflow "$@"
    ;;
  flower)
    sleep 10
    exec airflow "$@"
    ;;
  version)
    exec airflow "$@"
    ;;
  *)
    # The command is something like bash, not an airflow subcommand. Just run it in the right environment.
    exec "$@"
    ;;
esac
```

celery executor 를 사용하는 airflow 환경 구축을 알아봤습니다. 도커는 참 좋은 도구네요.
워커가 부족하다고 느껴질때는 worker를, 웹서버가 자주 죽는다면 2개를 띄워서 백업으로 쓸 수 있습니다.  

다음 명령어만 있으면 됩니다.  
`docker-compose -f docker-compose-CeleryExecutor.yml up -d --scale worker=10`

도커도 버전이 올라가다보니 도커컴포즈 버전별로 명령어들이 달라지기 시작하네요. 공부할게 많습니다.
