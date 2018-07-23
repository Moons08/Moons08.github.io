---
title: 오늘의 에러 - 180723 
date: 2018-07-23
tags: python Error
category: programming
---

해당 디렉토리에 있는 파일들의 확장자를 바꿔주는 코드를 실행하려다가 에러가 났다. 디렉토리 주소, 원 확장자, 바꿀 확장자 이렇게 세 개의 인자가 필요하다. 코드의 문제는 아니었고(내가 쓴 코드가 아니니까), 실행 방식의 문제였다.


분명 디렉토리 지정이 문제일 것이다 하고 `pwd`를 열심히 쳐가며 relative path로도 해보고, absolute path로도 해봤으나 문제는 다른 곳에 있었다.


## 내가 시도했던 코드

```bash
$ python test.py /home/mk/documents/dev/test, py, txt
FileNotFoundError: [Errno 2] No such file or directory: '/home/mk/documents/dev/test,'
```
결국 마지막에 붙어있는 **,** 를 보고 눈치챘다


## 올바른 코드

```bash
$ python test.py '/home/mk/documents/dev/test' py txt

$ python test.py /home/mk/documents/dev/test py txt
```

쉼표....

코드가 되는 이유는 하나인데, 코드가 안되는 이유는 무한하다는 말이 떠올랐다.
