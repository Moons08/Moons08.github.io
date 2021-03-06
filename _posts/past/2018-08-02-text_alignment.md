---
title: Text Alignment - 문자열 정렬
date: 2018-08-02
tags: python
category: programming
---

## python 문자열 정렬 기능
첫 번째 인자의 길이만큼 문자열을 할당하고, 문자열 제외 나머지 공간은 두 번째 인자(`default=' '`)로 채운다.

`str.ljust()` 왼쪽 정렬 <br>
`str.rjust()` 오른쪽 정렬 <br>
`str.center()` 가운데 정렬 <br>


```python
>>> width = 20
>>> string = 'A-yo'
>>> print(string.ljust(width, '-')) # 왼쪽 정렬
A-yo----------------
>>> print(string.center(width, '=')) # 가운데 정렬
========A-yo========
>>> print(string.rjust(width)) # 오른쪽 정렬
                A-yo

```

활용하면 다음과 같이 logo도 만들 수 있다.

```python
def make_logo(thickness):
    c = 'H'

    #Top Cone
    for i in range(thickness):
        print((c*i).rjust(thickness-1)+c+(c*i).ljust(thickness-1))

    #Top Pillars
    for i in range(thickness+1):
        print((c*thickness).center(thickness*2)+(c*thickness).center(thickness*6))

    #Middle Belt
    for i in range((thickness+1)//2):
        print((c*thickness*5).center(thickness*6))    

    #Bottom Pillars
    for i in range(thickness+1):
        print((c*thickness).center(thickness*2)+(c*thickness).center(thickness*6))    

    #Bottom Cone
    for i in range(thickness):
        print(((c*(thickness-i-1)).rjust(thickness)+c+(c*(thickness-i-1)).ljust(thickness)).rjust(thickness*6))
```
