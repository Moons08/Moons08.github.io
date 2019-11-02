---
title: OS - Virtual Memory
date: 2019-11-01
tags: CS
category: programming
toc: true
toc_sticky: true
mathjax: true
--- 

가상메모리란, 물리 메모리보다 큰 프로세스를 실행하기 위한 개념이다.
프로세스 이미지를 모두 메모리에 올리지 않고 현재 필요한 부분만 메모리에 올린다. <- 동적 적재와 유사(dynamic loading)

## 요구 페이지

프로세스 이미지는 backing store(=swap device)에 저장하고 지금 요구되는(필요한) 페이지만(Demand Paging) 메모리에 올린다. Valid 비트가 추가된 페이지 테이블을 하드웨어에서 지원해야 한다.

### Page Fault

접근하려는 페이지가 메모리에 없는 경우를 의미한다. 없으면? Interrupt error 때리고 OS가  backing store에서 페이지를 가져오게 한다. 해당 페이지 기록 후 page table의 valid 0 -> 1 후 작업 진행

- 용어
  - pure demand paging: 처음부터 계속 page fault 발생 -> 느리지만 메모리 절약 up
  - preparing: 미리 필요할 것이라 생각되는 부분을 챙겨서 들고옴

### Effective Access Time

유효 접근시간 계산 방법

- p: probability of a page fault = page fault rate
- $T_eff = (1-p) * T_m + p * T_p$  
(요구 페이지가 메모리에 있을 확률 \* 메모리속도 + 아닐 확률 \* swap device 속도)

### Locality of reference

- 메모리 접근은 시간적(반복문 존재), 공간적(block단위) 지역성을 가짐  -> 실제 페이지 부재 확률은 매우 낮다.
- HDD는 접근 시간이 너무 길다 -> SSD 또는 저가 DRAM을 swap device로 사용

## 페이지 교체

1. Demand Paging 으로 요구되는 페이지만 backing store에서 가져온다.
2. 프로그램이 실햄 됨에 따라 요구페이지는 늘어나고 언젠가 **메모리가 가득 차게** 됨.
3. 메모리가 가득 차도 추가 페이지는 필요하기 때문에 어떤 페이지(victim page)는 backing store로 치우고(page-out), 빈 공간에 추가 페이지를 가져온다(page-in)

### Victim page

I/O 시간 절약을 위해 이왕이면 **modify 되지 않은 페이지**를 victim으로 선택한다. modified 된 놈은 변경된 부분을 write (하드에) 해야하는데, 하드는 느림. modified 되지 않은 페이지는? 그냥 버리고 하드에서 read만 하면 됨 *modified bit (= dirty bit)*

### Page Replacement Algorithms

- page reference string (페이지 참조열)
  - CPU가 내는 주소: 100 101 102 432 612 103 104
  - page size = 100
  - 페이지 번호: 1 1 1 4 6 1 1 (100씩 묶음)
  - page reference string = 1 4 6 1 (1 한번 가져왔으니 또 가져올 필요 없음)
- page replacement algorithms
  - FIFO
  - OPT
  - LRU

#### First-In First-Out

먼저 올라온 페이지부터 하나씩 날리고 요구 페이지를 올림

- (당연히) 별로임
- Belady’s Anomaly
  1. 기본적으로 페이지 프레임의 수(=메모리 용량)를 늘리면 page fault 가 감소할 것으로 생각됨 (100개 보다 1000개씩 불러오면 요구페이지가 있을 확률이 높겠지?)
  2. 그러나 특정 경우에서 프레임 수 증가에 따라 Page Fault 수가 증가함을 발견

#### Optimal

가장 오랫동안 쓰지 않을 페이지 부터 날림

- 불가능 (just like SJF CPU scheduling Algorithm)

#### Least-Recently-Used

최근에 사용되지 않은 페이지부터 날림 (최근에 사용되지 않을 경우 나중에도 사용되지 않을 것이라 가정)

- 페이지 참조열 = 7 0 1 2 0 3 0 4 …
- num of frames = 3
  1. 7 0 1 까지는 fault 계속 발생 (메모리에 있는게 없음)
  2. 2 가 들어올 때 7 날림 (최근 사용 순서: 1-0-7)
  3. 0은 이미 메모리에 있음 (no fault)
  4. 3 들어올 때 0 아닌 1을 날림 (최근 사용 순서: 0-2-1)
  5. 0은 이미 메모리에 있음 (no fault)
  6. 4 들어올때 2 날림 (최근 사용 순서: 0-3-2)
  7. …

### Global vs. Local Replacement

|Global replacement|Local replacement|
|-—-|—--|
|메모리 상의 모든 프로세스 페이지에 대해 교체 가능|메모리 상의 자기 프로세스 페이지에 대해서만 교제|

global replacement가 더 효율적일 수 있음 -> 계속 사용되는 프로세스의 페이지 날리기 vs. 가끔 사용되는 프로세스의 페이지 날리기

## 프레임 할당

Frame = page, 페이지 단위로 프레임을 할당한다.

### Thrashing

<figure style="width: 450px"  class="align-right">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/img/os/VirtualMemory/thrashing.png" alt="">
  <figcaption>[Techniques to handle Thrashing - GeeksforGeeks]</figcaption>
</figure>

CPU utilization vs. Degree of multiprogramming  
프로세스의 개수가 증가하면 CPU 이용률이 증가하는데, 일정 범위를 넘어서면 CPU 이용률이 오히려 감소한다. 빈번한 page in/out 로 인해 i/o 시간이 증가하기 때문이다.

- 극복
  1. Global replacement 보다는 local replacement를 이용한다.
     - Why? -> Context switch 를 하면서 process끼리 서로 남의 페이지를 갖다 버리기 때문
     - *페이지교체 때와는 말이 다르다.*
  2. 프로세스당 **적절한** 메모리(프레임)을 할당한다.

### Frame Allocation

정적 할당과 동적할당으로 구분되며, 정적 할당에는 모든 프로세스에 동일한 프레임을 할당하는 균등 할당과 작업 사이즈에 비례하게 프레임을 할당하는 비례 할당이 있다. 그러나 전체 작업 사이즈와 실제 사용 사이즈에는 차이가 분명히 존재하기 때문에 동적 할당을 채택하게 된다. (Ex. word의 수많은 기능을 다 쓰지 않는데 다 할당하는 것은 비효율적이다.) 

#### Dynamic Allocation

- Working set model
  - 각 프로세스의 **시간대 별 프레임 사용량(working set)**을 이용하여 그 만큼의 프레임을 할당한다.
  - working set window: 얼마만큼의 시간범위(window)를 단위로 쓰는 지에 따라 결과가 달라진다.

<figure style="width: 450px"  class="align-right">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/img/os/VirtualMemory/PFF.png" alt="">
  <figcaption>[Techniques to handle Thrashing - GeeksforGeeks]</figcaption>
</figure>

- Page-Fault Frequency (PFF)
  - page fault 발생 비율의 상/하한선을 설정
  - 상한선 초과 프로세스에는 더 많은 프레임을 할당
  - 하한선 이하 프로세스의 프레임은 회수

### Page Size

페이지 크기는 기술 발전에 따라 점차 커지는 추세. 프로그램 사이즈도 커지고, 램사이즈도 커지고 있다.
페이지 사이즈의 영향은 평가 척도에 따라 다르다.
  
- 내부단편화
  - 페이지 크기가 작을수록 좋다 (남는 공간 X)
- page-in, page-out 시간
  - 페이지 크기가 클수록 좋다. (헤드 한번 움직여서 많이 불러옴)
- 페이지 테이블 크기
  - 페이지 크기가 클수록 좋다. 작아지면 테이블 크기(비용)가 커져야 한다.
- Memory resolution (필요한 것만 딱 갖고 있는 것을 의미)
  - 페이지 크기가 작을수록 좋다. (쓰지도 않는데 들고 있으면 안 좋음)
- Page fault 발생 확률
  - 페이지 크기가 클수록 발생 확률은 낮아진다.

### 기술 동향

초반에는 페이지 테이블을 위한 별도의 chip(TLB 캐시)이 있었으나 기술 발전에 따라 캐시 메모리, TLB 모두 on-chip, cpu 안에 넣어쓴다.
