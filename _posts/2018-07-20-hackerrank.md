---
title: coding quiz - ginortS
date: 2018-07-20
tags: algorithm
category: programming
---
알고리즘을 많이 풀어보지 않아서 해커랭크에서 쉬운 문제에도 고전하는 경우가 많은데, 난이도 medium임에도 불구하고 금방 풀어서 기념으로 기록

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
