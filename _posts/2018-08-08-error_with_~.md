---
title: 오늘의 에러 - 180808
date: 2018-08-08
tags: Error
category: programming
---
이번에도 경로(PATH) 문제였다.

docker를 이용해 mysql을 켜면서, host OS에 마운트하는 코드였다. (컨테이터가 제거되어도 안에 있는 데이터는 남기기 위한 마운트)


## 문제가 되었던 코드

```console
docker run -d -p 3306:3306 \
-e MYSQL_ALLOW_EMPTY_PASSWORD=true \
--name mysql \
-v /documents/test:/var/lib/mysql \
mysql:5.7
```
위 코드는 제대로 실행은 되는데 내가 만들어놓은 documents/test 안에 해당 데이터들이 하나도 들어가있지 않은 것이었다. 대체? 왜? 실행이 안됐으면 이해라도 할 텐데.
<br>
직접 터미널에서 `cd /documents/test`를 쳐보고 나서야 알게 되었다. 내 `home/user`, 안의 `documnets/test`가 아닌 가장 상위의 `/`안에 새 폴더를 만들어 버리고 거기에 마운트를 한 것이었다. *폴더가 없으면 새로 만들어서 마운트를 할 줄이야...* 편하긴 한데 나에게 혼란을 주었다.



<br>
## 올바른 코드

```console
docker run -d -p 3306:3306 \
-e MYSQL_ALLOW_EMPTY_PASSWORD=true \
--name mysql \
-v ~/documents/test:/var/lib/mysql \
mysql:5.7
```

`~`하나만 붙이면 되는 문제였다.
