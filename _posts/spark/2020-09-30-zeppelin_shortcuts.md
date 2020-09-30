---
title: Zeppelin notebook shortcuts
date: 2020-09-30
tags: Spark shortcut zeppelin
category: programming
toc: True
header:
  teaser: /assets/img/post/spark/zeppelin_classic_logo.png
sidebar:
    nav: "spark"
---

파이썬에 주피터 노트북이 있다면, 스파크에는 제플린 노트북이 있습니다. 이번에는 제플린 노트북에서 생산성을 극대화시킬 단축키에 대해 정리해보겠습니다. ~~Zeplin 아니고 Zeppelin~~

## 기본

![img](/assets/img/post/spark/zeppelin/shortcut_basic.png)

노트북 패러그래프 우측 상단의 톱니바퀴 모양을 누르면 다양한 단축키를 확인할 수 있습니다.  
width 조절은 `ctrl`+`alt`+`+/-` 로도 가능합니다.
> mac: `ctrl`+`shift`

## 에디터처럼

### 열 편집

![img](/assets/img/post/spark/zeppelin/edit_col.gif)

`ctrl` + `alt` + `아래위 방향키` 로 편집할 열을 선택. 아는 사람만 아는 열편집의 편리함..!
> mac: `ctrl`+`option`

### 범위 삭제

![img](/assets/img/post/spark/zeppelin/delete_range.gif)

`ctrl` + `d`
> mac: `command`+`d`

범위가 지정되어있지 않으면 라인 삭제. vscode에서는 이게 동일 부분 추가 선택(add selection to next find match)이라 헷갈리는 부분.

### 범위 복사

![img](/assets/img/post/spark/zeppelin/copy_range.gif)

`shift` + `alt` + `아래위 방향키` 로 선택된 범위를 통째로 복사.
> mac: `command`+`option`

### 동일단어 선택

![img](/assets/img/post/spark/zeppelin/add_select.gif)

`ctrl` + `alt` + `좌우 방향키` 로 선택된 범위와 동일한 부분을 추가로 선택. vscode의 ctrl + d
> mac: `ctrl` + `option`

### 숫자 변경

![img](/assets/img/post/spark/zeppelin/change_num.gif)

입력한 숫자 뒤에 커서를 놓고 `shift`+`ctrl`+`아래위 방향키`를 하면 숫자가 차례로 하나씩 올라가거나 내려갑니다. 음수와 실수도 지원합니다.
> mac: `shift`+`option`
