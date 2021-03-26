---
title: "[Operating System] Ch7. Deadlocks"
categories: "OS"
tags:
  - Operating System
  - Dead Lock
---

# Deadlock, 교착상태
![](/assets/images/study/dev/2021/os/ch7_deadlock.png)
교착상태<sub>deadlock</sub>의 개념을 그림을 통해 이해하면 쉽다.  
그림과 같이 각 차선에서 정해진 차선으로 이동하는 차들이 차선에 가득차서 이동이 불가능한 상태를 말하는데,  

이것을 프로그램 관점에서, 차선은 `컴퓨팅 자원`, 자동차는 `프로세스`라고 가정해보자.  
자원<sub>resources</sub>은 정해져 있는 상황에서 각각의 프로세스가 **컴퓨팅 자원**을 필요로 하는데, 서로 양보하지 않으며 **자원만 요청하고 있는 상황**으로 해석할 수 있다.

- Deadlock
  - 일련의 프로세스들이 서로가 가진 자원을 기다리며 `block`된 상태
- Resource
  - 하드웨어, 소프트웨어 등을 포함하는 개념

# Deadlock 발생의 4가지 조건
- **Mutual exclusion, 상호배제**
  - 매 순간 하나의 프로세스만이 자원을 사용할 수 있음
- **No preemption, 비선점**
  - 프로세스는 자원을 스스로 내어놓을 뿐 강제로 빼앗기지 않음
  - 외부에서 프로세스를 강제로 취득하지 않는 상황
- **Hold and wait, 보유대기**
  - 자원을 가진 프로세스가 다른 자원을 기다릴 때 보유 자원을 놓지 않고 계속 가지고 있음
  - 프로세스 입장에서 자신이 자원을 보유한 상태에서 자원을 취득하는 것을 기다리는 상황
- **Circular wait, 순환 대기**
  - 자원을 기다리는 프로세스간에 사이클이 형성되어야 함  
  아래 그림과 같이 프로세스간 서로의 자원을 기다리고 있는 상황
![](/assets/images/study/dev/2021/os/ch7_deadlock-prevention.jpeg)


## Resource-Allocation Graph, 자원할당 그래프
> 그래프내에 돌아오는 사이클이 있으면 Deadlock이 발생하는 상황이다.

- **사이클이 없는 상황**
  - ![](/assets/images/study/dev/2021/os/ch7_resource-allocation-graph.png)

- **사이클이 있는 상황**
  - 만약의 인스턴스가 1개밖에 없을때, 사이클이 발생한다면 **deadlock이 발생한다.**
  - 만약 인스턴스가 2개이상 있을때, 사이클이 발생한다면 **deadlock이 발생할수도, 발생하지 않을 수도 있다.**

아래 그림을 통해 deadlock 상황을 유추해보자.
  - ![](/assets/images/study/dev/2021/os/ch7_resource-allocation-graph-2.png)
    - 위 그래프 흐름을 보면,
      - R<sub>2</sub> -> P<sub>1</sub> -> R<sub>1</sub> -> P<sub>2</sub> -> R<sub>3</sub> -> P<sub>3</sub> -> R<sub>2</sub>
      - R<sub>2</sub> -> P<sub>2</sub> -> R<sub>3</sub> -> P<sub>3</sub> -> R<sub>2</sub>
    - 결국 이 사이클은 R<sub>2</sub>에 2개의 인스턴스가 존재하지만, 사이클이 2개 발생하고 있다. (**Deadlock 발생**)

  - ![](/assets/images/study/dev/2021/os/ch7_resource-allocation-graph-3.png)
    - 위 그래프 흐름을 살펴보면,
      - R<sub>2</sub> -> P<sub>1</sub> -> R<sub>1</sub> -> P<sub>2</sub>
      - R<sub>2</sub> -> P<sub>1</sub> -> R<sub>1</sub> -> P<sub>3</sub> -> R<sub>2</sub>
      - R<sub>2</sub> -> P<sub>4</sub>
    - 이와같이 각 인스턴스는 중복해서 프로세스의 요청을 받지 않으므로 **deadlock이 발생하지 않는다.**

# Deadlock 처리 방법
- Deadlock Prevention
  - 자원 할당 시 Deadlock의 4가지 필요 조건 중 하나가 만족되지 않도록 하는 것
- Deadlock Avoidance
  - 자원 요청에 대한 부가적인 정보를 이용해서 deadlock의 가능성이 없는 경우에만 자원을 할당
  - 시스템 state가 원래 state로 돌아올 수 있는 경우에만 자원을 할당
- Deadlock Detection and recovery
  - Deadlock발생은 허용하되 그에 대한 detection루틴을 두어 deadlock발견시 recover
- Deadlock Ignorance
  - Deadlock을 시스템이 책임지지 않음
  - UNIX를 포함한 대부분의 OS가 채택함
    - 사용자가 직접 프로세스를 강제 종료함

## Deadlock Prevention
- Mutual Exclusion
  - 공유해서는 안되는 자원의 경우 반드시 성립해야 함
- Hold and Wait
  - 프로세스가 자원을 요청할 때 다른 어떤 자원도 가지고 있지 않아야 한다.
    - 방법 1. 프로세스 시작 시 모든 필요한 자원을 할당받게 하는 방법
      - 모든 자원이 대기후에 시작하는 시점마다 보유하기 때문애 *비효율*
    - 방법 2. 자원이 필요할 경우 보유자원을 모두 놓고 다시 요청
      - 자원을 바로 사용하는 것이 아닌, 보유한(hold) 자원을 다시 내어 놓음(자진해서 반납)
- No Preemption
  - process가 어떤 자원을 기다려야 하는 경우 이미 보유한 자원이 선점됨
  - 모든 필요한 자원을 얻을 수 있을 때 그 프로세스는 다시 시작된다
  - 상태를 쉽게 저장하고 재사용할 수 있는 자원에서 주로 사용 (CPU, memory)
    - 예를들어 CPU의 경우 timer interrupt를 통해서 자원을 preemption하고 있음
- Circular Wait
  - 모든 자원 유형에 할당 순서를 정하여 **정해진 순서**대로만 자원 할당
    - 예를들어 순서가 3인 자원 R<sub>i</sub>를 보유 중인 프로세스가 순서가 1인 자원 R<sub>j</sub>을 할당받기 위해서는 반드시 R<sub>i</sub>를 release 해야 한다.


> **Deadlock Prevention의 한계**
Utilization 저하, throughput 감소, starvation 발생 가능

## Deadlock Avoidance
- 자원 요청에 대한 부가정보를 이용해서 자원 할당이 deadlock으로부터 안전한지를 동적으로 조사해서 안전한 경우에만 할당
  - 가장 단순하고 일반적인 모델은 프로세스들이 필요로 하는 각 자원별 최대 사용량을 미리 선언하도록 하는 방법
- safe state
  - 시스템 내의 프로세스들에 대한 `safe sequence`가 존재하는 상태
  - **safe sequence**
    - 프로세스의 sequence<P<sub>1</sub>, P<sub>2</sub>, ... , P<sub>n</sub>>이 safety하려면 P<sub>i</sub>(1<= i <=n)의 자원요청이 **가용 자원 + 모든 P<sub>j</sub>(j<i)의 보유 자원** 에 의해 충족 되어야 함
    - 조건을 만족하면 다음 방법으로 모든 프로세스의 수행을 보장
      - P<sub>i</sub>의 자원 요청이 즉시 충족될 수 없으면, 모든 P<sub>j</sub>(j<1)가 종료될 떄까지 기다린다
      - P<sub>i-1</sub>이 종료되면 P<sub>i</sub>의 자원요청을 만족시켜 수행한다

![](/assets/images/study/dev/2021/os/ch7_daedlock_avoidance.png)
- Deadlock avoidance의 2가지 알고리즘
  - 자원<sub>resource</sub>당 1개의 인스턴스만 존재할때
    - Resource Allocation Graph Algorithm 사용
  - 자원<sub>resource</sub>당 2개 이상의 인스턴스가 존재할때
    - Banker's Algorithm 사용


### Resource Allocation Graph Algorithm
![](/assets/images/study/dev/2021/os/ch7_resource-allocation-graph-algorithm.png)
다음 그래프의 선의 움직임을 설명하면
- 리소스(R)에서 나가는 **실선**은 자원을 할당
- 프로세스(P)에서 나가는 **실선**은 자원을 요청
- 프로세스(P)에서 자원(R)으로 나가는 **점선**은 프로세스가 적어도 한 번은 자원을 요청할 수도 있다.

위 그래프는 왼쪽에서 오른쪽의 흐름으로 진행하는 과정을 나타낸 건데, 해석해보면  
- P<sub>2</sub>는 처음 R<sub>1</sub>에 자원을 요청을 하며, R<sub>2</sub>에 요청을 하기도 한다. 
- R<sub>1</sub> 자원은 P<sub>1</sub>에게 자원을 할당해주기 때문에 P<sub>2</sub>는 계속해서 기다리다가 어떤 특정 시점에 R<sub>2</sub>에게 자원을 요청하여 R<sub>2</sub>자원을 할당 받는다.

위 흐름은 `daedlock`이 발생하지 않는 평화로운 상황이다.  

그런데 갑자기 P<sub>1</sub>에서 R<sub>2</sub>로 향하는 점선이, 실선이 되면서 자원을 요청한다면,  
위 그래프는 전체적으로 사이클을 이루는 모양이 되면서 `daedlock`이 발생하게 될 것이다.

즉 그래프처럼 프로세스가 당장 자원을 쓰지 않는다고해서 필요한 시점에 자원을 내어 주게 된다면, 그래프의 점선이 실선이 되며 사이클을 형성하여 데드락을 유발시키게 된다.  

**daedlock avoidance**는 이런 사이클을 미연에 방지하고자 애초에 자원이 할당될 경로를 차단하는 것이다.  
![](/assets/images/study/dev/2021/os/ch7_r-a-g-reject.png)

### Banker's Algorithm
![](/assets/images/study/dev/2021/os/ch7_bankers_algorithm.png)


---

*Reference*

- Operating System Concepts (Paperback / 9th Ed.) Books