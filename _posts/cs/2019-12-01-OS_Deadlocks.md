---
title: OS - Deadlocks
date: 2019-12-01
tags: CS
category: programming
toc: true
toc_sticky: true
--- 

프로세스는 작업을 진행하기 위해 자원을 필요로 하는데, 한정된 자원을 때문에 프로세스가 진행되지 못하고 무한정 대기하게 되는 교착상태가 발생하기도 한다.

1. A 프로세스가 a 자원을 가진 상태에서 b자원을 위해 대기를 하는 중이고
2. B 프로세스는 b 자원을 가지고 a 자원 대기 중이라면
3. 교착상태가 발생할 가능성이 존재한다.

## 자원

자원은 요청(request) -> 사용(use) -> 반납(release)의 과정으로 사용된다. 이는 자원할당도 (Resource Allocation Graph)로 표현할 수 있는데, 할당도에 원이 만들어지면 교착 상태의 필요조건 하나가 충족된다. 프로세스가 필요로 하는 동일 형식 자원이 여러개 있을 수 있다. (CPU 2개, 프린터 3개 등)

## 교착상태 필요조건

모두 충족해야 한다. (필요조건이니까)

- mutual exclusion: 프로세스간 자원을 공유할 수 없다
- hold and wait: 프로세스가 자원을 보유하고 그 상태로 대기
- no preemption: 들고 있는 자원을 뺏을 수 없다
- circular wait: 프로세스들이 환형으로 대기 (A -> B -> C -> A )

## 교착상태 처리

교착상태를 처리하기 위해서 4가지 방법이 존재한다. (방지, 회피, 검출 및 복구, 무시)

### 교착상태 방지

4가지 필요조건 중 한가지만 불만족 시키면 된다.

- mutual exclusion: 원천적으로 불가능한 경우가 많다. (읽기만 하는 파일 정도?)
- **hold and wait**: 프로세스가 자원을 가지고 다른 자원을 기다리지 않도록 한다. 즉, 강제로 자원을 release하게 한다. 하지만 자원 활용률이 저하되고 starvation 문제가 발생할 수 있다. (starvation: 자원 할당 문제로 영원히 완료되지 못함)
- no preemption: 일반적으로는 불가능하다.
- **circular wait**: 자원에 번호를 부여하고, 오름차순으로만 자원을 요청하도록 한다. 자원 활용률이 저하되는 단점이 있다.

### 교착상태 회피

교착상태를 자원 요청에 대한 잘못된 승인으로 인식한다. 현재 요구하는 자원뿐만 아니라 프로세스가 최대로 요구하는 자원을 고려하여 할당한다 (safe allocation) *bank run에 대비한 준비금 정도로 이해할 수 있을 듯 하다- Banker's Algorithm*

### 교착상태 검출 및 복구

교착 상태가 일어나는 것을 원칙적으로 허용한다. 이후 주기적 검사를 통해 교착상태가 발견되면 그 부분에 대해 복구한다. 복구는 프로세스를 일부 강제 종료하거나 자원을 강제로 선점하여(뺏어서) 일부 프로세스에 할당한다. 검출 과정의 계산 및 메모리 부담(overhead)이 발생할 수 있다.

### 교착상태 무시

교착 상태는 실제로 잘 일어나지 않기 때문에, 그냥 무시한다. 방지, 회피, 검출처럼 다른 처리를 위해 성능을 떨어뜨리는 일을 하지 않는다. *오류나면? reboot*
