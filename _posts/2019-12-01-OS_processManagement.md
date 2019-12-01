---
title: OS - Process Management
date: 2019-12-01
tags: CS
category: programming
toc: true
toc_sticky: true
--- 

프로그램은 디스크 안에 잠자고 있는 상태이고, 프로그램을 메모리에 올리면(실행시키면) 프로세스가 된다.  

프로세스 생애주기 예시

1. **New**: main memory 입성
2. **Ready**: 실행 준비 완료
3. **Running**: 실행
    1. Print 요청 -> I/O -> waiting
    2. Print 작업 완료 -> 2. ready -> 3. running
4. **Terminate**: 작업 완료

## Process Control Block (PCB)

프로세스에 대한 모든 정보를 가지고 있다.  다음과 같은 정보들을 포함한다.
상태정보(ready, waiting, running, terminate), MMU info(base, limit), registers, CPU time, process id(PID), ...

## Queues

<figure style="width: 500px"  class="align-center">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/img/os/ProcessManagement/process.jpg" alt="">
  <figcaption></figcaption>
</figure>

각각의 queue(대기라인)에는 저마다의 scheduler가 존재한다. scheduler는 어떤 프로세스를 언제 어떻게 처리할지 결정한다.

Job Queue
: 잠에서 깨어난 프로세스들이 cpu에서 처리되기 위해 기다리고 있는 줄이다. Long-term scheduler인 Job scheduler가 이 줄을 처리한다.

Ready Queue
: 상대적으로 빠르고 자주 일을 하는 CPU scheduler(Short-term scheduler)가 큐를 관리한다. cpu에서 처리되다가 할당된 시간이 지난 프로세스는 다시 ready queue로 돌아가 순서를 기다리게 된다.

Device Queue
: 프린터, 마우스, 디스크 등의 device들에 대한 요청을 처리하는 device scheduler가 관리하는 큐. cpu에서 프로세스 연산을 하다가 disk에 결과물을 써야하는 상황이 오면 disk queue로 가서 i/o 작업을 마치게 되고 , 다시 ready queue로 돌아가 자기 차례를 기다리게 된다.

## Multiprogramming

여러개의 프로그램들이 단일 프로세서에서 동시에 실행되는 것이다. 다만, 실제로 동시에 실행되는 것은 아니고 CPU가 굉장히 빠르게 전환하며(context switching) a, b, c, a, b, c 식으로 작업을 하기 때문에 동시에 **실행되는 것처럼** 보인다.

Degree of multiprogramming
: 메인 메모리에서 처리되고 있는 프로세스의 개수를 말한다.

I/o-bound vs. cpu-bound process
: I/o 빡세게 쓰는 프로세스가 있는 반면, cpu를 많이 쓰는 프로세스가 있다. Job scheduler가 이런 애들을 적절히 분배해서 작업을 해야한다.

Swapping
: 유휴 자원(계속 놀고있는 프로세스)을 지켜보고 있다가 main memory에서 swap device로 내쫓고(swap out), 필요해지면 다시 불러온다. (swap in)

### Context switching

Scheduler
: 다음엔 어떤 프로세스를 실행할지 결정

Dispatcher
: 다음에 돌아왔을 때 중단 지점부터 작업을 이어갈 수 있도록 프로세스의 현재 상태를 PCB에 save & restore

Context switching overhead
: 당연히 낮아야 좋기 때문에 저수준의 assembler comes in
