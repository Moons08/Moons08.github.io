---
title: OS - File Allocation
date: 2019-10-28
tags: CS OS
category: programming
toc: true
toc_sticky: true
mathjax: true
---  
보조기억장치인 파일 시스템(하드 디스크)에 파일을 할당하는 방법

## 하드디스크

- track(cylinder)과 sector로 구성
- Sector size = 512 bytes
- **블록 단위**로 읽기/쓰기
- 디스크를 pool of free blocks 라고 함

## 파일할당

- Contiguous Allocation 연속 할당
- Linked Allocation 연결 할당
- Indexed Allocation 색인 할당

### Contiguous Allocation

<figure style="width: 160px"  class="align-right">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/img/os/FileAllocation/contiguous_index.png" alt="">
  <figcaption>contiguous index</figcaption>
</figure>

디스크 상의 연속된 블록에 파일 할당

- 장점
  - 디스크 헤더 이동 최소화 (빠른 I/O)
  - sequential, direct access 가능
- 단점
  - 파일 삭제 시
    - 홀 생성 → **외부 단편화** → 공간 낭비 → compaction 비용 추가
  - 파일 생성 시
    - 파일 크기 가늠 X → 어디에 놓을지 모르게 됨
    - 파일 크기가 계속 증가할 경우? Ex) *log file* → 문제  

### Linked Allocation

<figure style="width: 250px"  class="align-right">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/img/os/FileAllocation/linked_index.png" alt="">
  <figcaption>linked index</figcaption>
</figure>

파일 블록마다 다음 블록을 가리키는 **포인터를 저장**

- 장점
  - 외부 단편화 해결
- 단점
  - Direct access 불가능  
        *Skip 그런거 없어→* 처음부터 다 읽어야함
  - 포인터
    - 저장 공간 손실 (min. 4 byte)
    - 포인터 유실되면? *노답* → 낮은 신뢰성
  - 디스크 헤더가 왔다갔다 해야함 → 속도저하 *(SSD라면 어떨까?)*

#### 향상: FAT 파일 시스템 (File Allocation Table)

<figure style="width: 300px"  class="align-right">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/img/os/FileAllocation/linked_FAT.png" alt="">
  <figcaption>file allocation table</figcaption>
</figure>

포인터만 모은 테이블을 **별도 블록** 에 저장 (손실 복구를 위해 이중 저장)

- Direct access 가능 → FAT만 읽으면 됨
- 일반적으로 FAT는 메모리에 올려 사용한다(Caching) *빨리빨리*
  - 주기적으로 FAT에 대한 변동을 체크

- FAT32, FAT16 숫자의 의미?
  - 숫자는 FAT 인덱스 하나가 저장할 수 있는 숫자 크기를 의미
    즉, FAT32(32bit)는  $2^{32}$ 까지의 인덱스 저장 가능  
    (**더 큰 파일**은 어떻게 하지?)
- MS-DOS, Windows 등에서 사용

---

### Indexed Allocation

<figure style="width: 200px"  class="align-right">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/img/os/FileAllocation/indexed.png" alt="">
  <figcaption>indexed allocation</figcaption>
</figure>

파일 마다 포인터의 모음인 인덱스 블록을 하나씩 생성. 디렉토리는 인덱스 블록을 가리킴

- 장점
  - Direct access 가능
  - 외부 단편화 X
- 단점
  - 인덱스 블록만큼의 공간 손실
  - 블록 크기에 따라 파일의 최대 크기 제한
    - Ex) 1 블록이 512 byte = 4 byte \* 128개 인덱스 -> 128 \* 512 byte = 64KB  
      *이걸 어따 붙여* -> 인덱스 블록 여러개 사용  

#### 향상: Linked, Multilevel index, Combined

인덱스 블록을 여러개 사용하여 단점 극복. Unix/Linux 등에서 사용

<figure style="width: 600px"  class="align-center">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/img/os/FileAllocation/indexed_linked.png" alt="">
  <figcaption>linked index</figcaption>
</figure>  

<figure style="width: 300px"  class="align-left">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/img/os/FileAllocation/indexed_multilevel.png" alt="">
  <figcaption>multilevel index</figcaption>
</figure>  

<figure style="width: 300px"  class="align-right">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/img/os/FileAllocation/indexed_combine.png" alt="">
  <figcaption>combined index</figcaption>
</figure>