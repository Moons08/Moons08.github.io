---
title: hackerrank - ginortS
date: 2018-07-20
tags: algorithm
category: programming
---
해커랭크에서 쉬운 문제에도 고전하는 경우가 종종 있는데, 자주자주 이렇게 푸는 습관을 가져야겠다.

## task
You are given a string which contains alphanumeric characters only
1. All sorted lowercase letters are ahead of uppercase letters.
1. All sorted uppercase letters are ahead of digits.
1. All sorted odd digits are ahead of sorted even digits.


```python
givenString = str(input()) # Sorting1234

lower = []
upper = []
odd = []
even = []

for i in sorted(givenString):
    try:
        if int(i) % 2 == 0:
            even.append(i)
        else:
            odd.append(i)
    except:
        if i.upper() == i:
            upper.append(i)
        else:
            lower.append(i)


print(''.join(lower+upper+odd+even) # ginortS1324
```
