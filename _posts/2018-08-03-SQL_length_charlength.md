---
title: [SQL] LENGTH vs CHAR_LENGTH
date: 2018-08-03
tags: SQL
category: programming
---

안 쓰던 SQL을 연습하면서 퀴즈를 풀다 보니, 한가지 의문이 생겼다. length나 char_legth나 같은 값을 내는 것 같은 데, 왜 두 개나 있을까? 예전부터 쓰던 문법이라 그런 것인가 싶었지만 다른 이유가 있었다.


- LENGTH()는 문자열 길이를 bytes 단위로 반환한다.
- CHAR_LENGTH()는 문자열 길이를 character 단위(우리가 흔히 쓰는 단위)로 반환한다.


굳이 나눠놓은 이유는 문자열에 대한 인코딩 방식 때문인 것 같다. 유니코드에서 대부분의 문자는 2바이트의 크기로 인코딩 되는데, 인코딩 방식은 유니코드 뿐 아니라 굉장히 많고(...) 언제나 예외는 발생한다.  


아래는 그런 상황의 예시.

```SQL
select length(\_utf8 '€'), char_length(\_utf8 '€')
--> 3, 1
```
유로 사인은 1글자(character)이지만 3 byte를 차지한다. (encoded as 0xE282AC in UTF-8)
