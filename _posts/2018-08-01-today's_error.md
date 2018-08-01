---
title: round(3.5) == round(4.5) 는 왜 True 일까?
date: 2018-08-01
tags: python Error
category: programming
mathjax: true
---

해커랭크를 풀면서 발생한 에러다. 문제를 풀던 중, 홀수의 숫자를 반으로 나눈 값의 몫 + 1의 값이 필요했다. 예를 들어 7을 받으면 4, 9를 받으면 5를 반환하면 되었다. 3.5나 4.5나 반올림하면 4, 5 이런 식으로 나오니까 `round`를 썼다. 결과는 에러.

```python
>>> round(7/2)
4
>>> round(9/2)
4
>>> round(7/2) == round(9/2)
True
```

## 해결 방법

```python
>>> import math
>>> math.ceil(7/2)
4
>>> math.ceil(9/2)
5
>>> 7//2 + 1
4
>>> 9//2 + 1
5
```

## 원인
파이썬 공식 문서 에서는 round 함수에 대해 다음과 같이 설명하고 있다.
>The behavior of round() for floats can be surprising: for example, round(2.675, 2) gives 2.67 instead of the expected 2.68. This is not a bug: it’s a result of the fact that most decimal fractions can’t be represented exactly as a float. See Floating Point Arithmetic: Issues and Limitations for more information.

즉, **부동 소수점(floating point)** 문제라고 한다. 컴퓨터는 10진수가 아닌 2진수를 사용하여 연산하는데, **10진 소수는 정확하게 2진수로 표현될 수 없다.** 따라서 일반적으로 입력하는 십진 부동 소수점 숫자가 실제로 기계에 저장될 때는 이진 부동 소수점 수로 **근사하게** 된다. 이처럼 부동 소수점으로 표현한 수가 실수를 정확히 표현하지 못하는 것은 위와 같이 다양한 문제를 낳는다.
