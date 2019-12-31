---
title: OS - Disk Scheduling
date: 2019-10-28
tags: CS
category: programming
toc: true
toc_sticky: true
mathjax: true
--- 
디스크 접근 시간은 Seek time, rotational delay, transfer time 으로 구성되며, 이 중 데이터의 위치는 찾는 Seek time 이 가장 오래 걸린다.
데이터를 읽고 쓰는 작업을 위한 대기열인 디스크 큐 (disk queue) 에는 많은 요청(request)이 쌓여있다.
디스크 스케쥴링은 이렇게 쌓여있는 요청들을 효율적으로 처리하는 방법이다.

## Basic

### FCFS

<figure style="width: 300px"  class="">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/img/os/DiskScheduling/FCFS.png" alt="">
  <figcaption>갈팡질팡</figcaption>
</figure>

First-Come First-Served. 먼저 온 작업 부터 실행한다. 그냥 봐도 효율적이지 않다.
그림의 head는 53번에 위치하고, Disk queue는 98, 183, 37, 122, 14, 124, 65, 67 인 경우

### SSTF

<figure style="width: 300px"  class="">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/img/os/DiskScheduling/SSTF.png" alt="">
  <figcaption>조금 나아졌다</figcaption>
</figure>

Shortest-Seek-Time-First. 현재 헤드 위치에서 가장 가까운 작업부터 실행한다. 하지만 아무리 기다려도 차례가 오지 않는 작업이 생길 수 있다. 
(Starvation)

## Developed

<figure style="width: 450px"  class="align-right">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/img/os/DiskScheduling/Elevator.png" alt="">
  <figcaption>헤드의 움직임이 건물 엘리베이터의 움직임 같다고 해서 Elevator Algorithms 라고 이름 붙여졌다</figcaption>
</figure>

### SCAN

디스크 헤드가 지속해서 앞뒤로(처음부터 끝까지) 디스크를 전체를 탐색한다. 탐색하는 과정에 해당 작업 데이터를 맞닥뜨리면 처리하는 식.
다만 방금 탐색한 구간을 되돌아가며 탐색하는 시간이 허비된다. (그새 작업이 쌓일 수는 있지만, 희박하다) 그래서 Circular SCAN이 등장

### C-SCAN

디스크 실린더를 원통(Circular)이라 생각하고(끝과 끝이 연결) 헤드가 전체를 순환 탐색한다. (0, 1, ..., 199, **200, 0**, 1, 2, ...,199)

### LOOK

헤드가 딱 마지막 요청(final request)까지만 보고(LOOK) 반대쪽으로 돌아간다. 아무 요청도 없는 부분을 위해 끝까지 확인하지 않는다.

### C-LOOK

LOOK에 Circular 개념 추가
