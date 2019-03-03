---
title: \[Airflow] 간단 설치
date: 2019-03-01 09:00:00 -0600
tags: airflow
category: programming
---

다양한 extra pakages가 존재하나, 일단 기본 airflow를 설치합시다.

```sh
$ pip install apache-airflow
```

```
RuntimeError: By default one of Airflow's dependencies installs a GPL dependency (unidecode).
To avoid this dependency set SLUGIFY_USES_TEXT_UNIDECODE=yes in your environment when you install or upgrade Airflow.
To force installing the GPL version set AIRFLOW_GPL_UNIDECODE
```
위와 같은 에러가 나온다면 다음 명령어로 설치해보자

    $ AIRFLOW_GPL_UNIDECODE=yes pip3 install apache-airflow

설치가 완료되면,

    $ airflow initdb

이러면 홈 폴더 아래 airflow 폴더가 생성됩니다.

 ```sh
 ~/airflow$ tree
 .
├── airflow.cfg
├── airflow.db
├── logs
│   └── scheduler
│       ├── 2019-02-28
│       └── latest -> /home/mk/airflow/logs/scheduler/2019-02-28
└── unittests.cfg

4 directories, 3 files
```

여기에 python 스크립트들이 들어갈 `dags` 폴더 하나 만들어 줍시다. 설치 끝입니다.
