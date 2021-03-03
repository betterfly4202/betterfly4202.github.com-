---
title: "[Operating System] Ch5. CPU Scheduling"
categories: "OS"
tags:
  - Operating System
  - CPU
  - CPU Scheduling
---

# CPU Scheduling
![](/assets/images/study/dev/2021/os/ch5_cpu_io_execution.png)
프로그램은 그림과 같이 CPU를 사용하여 instruction을 실행하는 단계(`CPU burst`)와 I/O를 실행하는 단계(`I/O burst`)의 연속으로 진행된다.  
즉 **어떤 프로그램이든 CPU burst 와 I/O burst를 반복하며 실행된다.**

![](/assets/images/study/dev/2021/os/ch5_cpu-burst-time.png)
여러 종류의 job(=process)이 섞여 있기 때문에 CPU 스케줄링이 필요
- Interactive job에게 적절한 response 제공 요망
- CPU와 I/O 장치 등 시스템 자원을 골고루 효율적으로 사용

그림을 보고 오해를 하면 안될 것이,  
I/O바운드 잡의 그래프 기울기가 높은것으로 CPU를 많이 사용하는 것인가 라고 생각이 될 수 있다.  
I/O바운드 잡은 CPU를 많이 사용하는 것이 아니라, CPU를 짧게 사용하는데, I/O controller와 매우 빈번하게 왔다갔다 하기 때문에 **빈도가 잦은 것**이다.

**CPU 스케줄링이 필요한 이유는,**  
I/O 바운드 잡이 문제인데, I/O 잡은 사람 또는 외부와 interaction이 계속해서 일어나는데, CPU를 획득하지 못하면 CPU를 사용할 수 있을때까지 무한정 대기할 수 밖에 없어진다.  
때문에 사람 또는 외부와 interaction에 사용되는 I/O bound job을 실행시 CPU 사용을 우선적으로 전달하여, 사용자가 느끼는 대기 시간을 감소 시켜주는 것이 CPU 스케줄링의 역할 중 하나이다.


- I/O Bound job
  - CPU를 잡고 계산하는 시간보다는 I/O에 많은 시간이 필요한 Job *(many short CPU bursts)*
- CPU bound job
  - 계산 위주의 Job *(few very long CPU bursts)*

## CPU Scheduler & Dispatcher  
### CPU Scheduler  
- Ready 상태의 프로세스 중에서 이번에 CPU를 줄 프로세스를 고른다.(프로세스를 선택하는 과정)  
  (CPU Scheduler라는 것이, 별도의 하드웨어나 소프트웨어가 하는 일이 아니다. OS안에 내재되어 있는 **코드 영역**을 말한다.)

### Dispatcher  
- CPU의 제어권을 CPU scheduler에 의해 선택된 프로세스에게 넘긴다. (실제로 CPU를 넘기는 과정)
- 이 과정을 `context switch`라고 한다

### CPU 스케줄링이 필요한 경우는 프로세스에게 다음과 같은 상태 변화가 있는 경우(예시)  
1. Running -> Blocked (예: I/O요청하는 시스템 콜)
   - CPU를 점유해봐야 I/O작업으로 인해 CPU활용이 불필요한 경우
2. Running -> Ready (예: 할당시간 만료로 timer interrupt)
   - 무한정 CPU를 사용하도록 할 수 없기 때문에, 특정 시간 이후로 CPU 점유를 빼앗는 경우
3. Blocked -> Ready (예: I/O완료 후 인터럽트)
   - I/O 작업이 끝난 후 Device Controller가 Interrupt를 걸어서 ready 상태로 변경하여 CPU 사용 준비를 알림
4. Terminate
   - 프로세스가 종료 후 새로운 프로세스에게 CPU를 넘겨주는 경우

> 1,4 스케줄링은 CPU를 갖고 있어도 사용할 필요가 없기 때문에 **자진 반납하는경우** `nonpreemptive`  
> 그 외 모든 스케줄링은 **강제로 CPU점유를 빼앗음** `preemptive`

## Scheduling Criteria
**Performance Index(= Performance Measure, 성능 척도)**

### 시스템 관점
시스템입장에서는 하나의 CPU자원으로 최대한 일을 많이 시키는 것이 효율적임(CPU utilization, Throughput)
#### CPU Utilization(이용률)
- Keep the *CPU as busy as possible*
- 전체 시간 중에 CPU가 사용되는 시간의 비율

#### Throughput(처리량)
- *#of processes* that *complete* their execution per time unit
- 주어진 시간안에 몇 개의 일(작업)을 해냈는지의 수치

### 프로세스 관점(사용자 관점)
CPU를 빠르게 획득하여 짧은 시간안에 프로세스를 마치는 것에 초점(Turnaroung time, Waiting time, Response time)
#### Turnaround time(소요시간, 반환시간)
- amount of time to *execute a particular process*
- CPU를 모두 사용한 후 I/O를 빠져나갈때까지의 시간

#### Waiting time(대기시간)
- amount of time a process has been *waiting in the ready queue*
- CPU를 쓰기 위해서 기다리는 시간(Ready queue에서 CPU작업을 대기)

#### Response time(응답 시간)
- amount of time it takes *from when a request was submitted until the first response is produced.* **not** output
- Ready Queue에서 CPU를 얻기까지 걸리는 시간
- *for time-sharing environment*


#### Waiting time과 Response time의 비교

![](/assets/images/study/dev/2021/os/ch5_Preemptive-Scheduling-vs-Non-Preemptive-Scheduling.png)
선점형 스케줄링`preemptive scheduling`의 경우 CPU를 획득 후 interrupt에 의해 CPU를 뺏기고 획득하고의 과정을 반복한다.(상단 그림 참고)  
이 경우 한번의 CPU burst기간에 CPU를 여러차례 선점하거나 양도하는 과정에서 많은 `대기시간`이 발생한다. 이떄 발상하는 모든 대기시간을 합친 시간을 `waiting time`이라고 한다.  

`Response time`은 이 과정에서 처음으로 CPU를 획득하기 까지 걸리는 시간을 의미한다.

### Scheduling Criteria examples
#### 중국집을 운영하면서 주방장을 한명 고용했다 => CPU 1개

- 사장입장에서는 주방장이 근무시간 중에 놀지않고 **최대한 많은 시간** 일을 해줬으면 좋겠다 -> `CPU Utilization`
- 중국집에 단위 시간당 최대한 **많은 양**의 손님이 방문하면 좋겠다 -> `Throughput`
- 중국집의 고객입장에서, 식사가 나오면 다 먹고 나갈때까지 걸리는 시간 -> `Turnaround time`
- 중국집에서 손님이 밥을 먹는 시간이 아닌, 식사가 나오기 전까지 기다리는 시간 -> `Waiting time`
- 첫번째 음식이 나올때까지 기다리는 시간 -> `Response time`

## Scheduling Algorithms
### FCFS(First-Come First-Served)
먼저 온 *순서대로* 처리한다.

- 프로세스의 도착순서 P1, P2, P3 스케줄 순서를 `Gantt Chart`로 보면 다음과 같다.
![](/assets/images/study/dev/2021/os/ch5_fcfs.png)

- Waiting time P1 = 0, P2 = 24, P3 = 27
- Average waiting time : (0+24+27)/3 = **17**

하지만, 프로세스의 도착 순서가 다음과 같이 된다면 어떨까?  
P2 -> P3 -> P1

![](/assets/images/study/dev/2021/os/ch5_fcfs_convoy_effect.png)

- Waiting time P2 = 0, P3 = 3, P1 = 6
- Average waiting time : (0+3+6)/3 = **3**

이 전의 순서에 비해 월등히 짧은 waiting time을 갖게 된다. 
이처럼 `FCFS`는 어떤 순서냐에 따라 기다리는 시간이 상당히 영향받는 것을 볼 수 있다.

이처럼 긴 대기를 부르는 프로세스를 수행하느라, 상대적으로 짧은 시간의 프로세스에 영향을 주는 현상을 `Convoy Effect`라고 한다.

### SJF(Shortest-Job-First)
- 각 프로세스의 다음번 `CPU burst time`을 가지고 스케줄링에 활용
- CPU burst time이 가장 짧은 프로세스를 제일 먼저 스케줄
- Two schemes:
  - Nonpreemptive
    - 일단 CPU를 선점하면 이번 CPU burst가 완료될 때까지 CPU를 선점(preemption)당하지 않음
    - ![](/assets/images/study/dev/2021/os/ch5_sjf_non-preemptive.png)
    - Average waiting time = (0+6+3+7)/4 =4
    - **CPU를 다 쓰고 나가는 시점에 다음 프로세스의 스케줄링을 결정한다**
  - Preemptive
    - 현재 수행중인 프로세스의 남은 burst time보다 **더 짧은 CPU burst time**을 가지는 새로운 프로세스가 도착하면 CPU를 빼앗김
    - 이 방법을 `Shortest-Remaining-Time-First(SRTF)`라고 부름
    - ![](/assets/images/study/dev/2021/os/ch5_sjf_preemptive.png)
    - arrival time이 0인 P1부터 CPU를 얻었지만, 더 짧은 수행시간(burst time)의 프로세스가 들어오면 CPU를 양도한다. 
    때문에 2.0초가 지난 시점에 도착한 P2의 수행속도는 4초 이므로, P1은 전체 7초에서 2초를 사용했기 때문에 5초, P2는 4초 이므로 burst time이 짧은 P2에게 CPU를 양도한다.
    - 이런식으로 매 시점마다 CPU를 선점적하여 가장 짧은 수행시간을 순서대로 스케줄링이 되면 다음과 같은 `Waiting time`이 발생한다.
    - Average waiting time = (9+1+0+2)/4 = 3
    - **새로운 프로세스가 도착할때마다 스케줄링을 결정한다.**
  - **SJF is optiomal**
    - 주어진 프로세스에 대해 *minimum average waiting time*을 보장

#### SJF의 문제점
이렇게 **Preemptive SJF** 알고리즘을 이용하면 최선의 효율적인 스케줄링을 할 수 있을 것 같은데, 여기 두가지 문제점이 존재한다.
1. SJF는 극단적으로 CPU사용 시간이 짧은 잡을 선호한다. 때문에 CPU 사용 시간이 긴 프로세스는 영원히 CPU할당을 받지 못하게 될 수 있다. `Starvation`
   - 예를들어 CPU 1초 짜리 프로세스 `A`와 10초짜리 `B`가 있는데, A종료 후 B를 실행하려는데 그 사이에 2초짜리 프로세스가 끼어들고... 이런식으로 계속해서 중간에 짧은 시간의 프로세스가 계속해서 개입된다면 가장 긴 수행시간의 프로세스는 영원히 실행되지 못할수도 있다.
2. CPU 사용시간을 미리 알기가 어렵다.
   - 프로그램 내부의 다양한 로직들과 연결 단계가 있는데, 이것을 실행하지 않고 ready queue상태에서 프로세스의 수행시간을 정확히 예측하는 것이 어렵다.

이처럼 미리 사용시간을 예측하는 것은 어렵지만, 비슷한 잡들의 과거 수행시간을 견주어봐서 *대략적인* 수행시간을 예측해서 스케줄링에 반영할 수 있다.

### Priority Scheduling
- 우선순위가 높은 프로세스에게 CPU를 할당한다. (**smallest integer = highest priority**)
  - Preemptive
    - 우선순위가 높은 프로세스에 CPU를 할당했는데, 더 높은 우선순위의 프로세스가 넘어온다면 CPU를 넘겨준다.
  - Nonpreemptive
    - 일단 CPU를 할당했으면, 더 높은 우선순위의 프로세스가 오더라도 이전에 수행중이던 프로세스가 완료된 후 CPU를 넘겨준다.
- SJF는 일종의 priority scheduling이다.
  - priority = predicated next CPU burst time
- 문제점
  - `Starvation` : 가장 낮은 우선순위의 프로세스는 영원히 실행되지 않을 수 있음
- 해결책
  - `Aging` : 우선순위가 낮은 프로세스라도, 대기시간이 길어지면 우선순위를 점차 높여주는 방식

### Round Robin(RR)
- 각 프로세스는 동일한 크기의 할당시간<sub>time quantum</sub>을 가짐 (*일반적으로 10~100 milliseconds*)
- 할당시간이 지나면 프로세스는 선점<sub>preempted</sub>당하고 ready queue의 제일 뒤에 다시 줄을 서게 된다.
  - 어떤 프로세스든지 공평한 대기시간을 갖기 때문에, 앞서 살펴봤던 `Scheduling Criteria`의 **Response Time**이 짧다.  
    조금씩 CPU를 줬다 뺏었다를 반복하기 떄문에 어떤 프로세스라도 짧은 시간안에 CPU를 할당받을 수 있는 기회가 주어진다.
- n개의 프로세스가 ready queue에 있고, 할당 시간이 `q time unit`인 경우 각 프로세스는 최대 q time unit단위로 CPU시간의 **1/n**을 얻는다.
  - 어떤 프로세스다 (n-1)q time unit 이상 기다리지 않는다.
- Performance
  - q large(할당시간을 길게 잡는 경우) => FCFS
  - q small(할당시간을 짧게 잡는 경우) => 빠른 순환을 하는데는 이상적이지만, context switch가 빈번히 발생하여 오버헤드가 커짐

**Round Robin에서 한가지 유의할 점은,**  
만약 실행시간 **100초 짜리 프로세스가 4개**가 있고, **time quantum을 1초**로 설정했다고 가정하자.  
각 프로세스는 CPU를 받고, 1초씩 실행되면서 context switch가 발생하며 프로세스가 실행된다. 이때 context switch 시간을 제외하더라도 4개의 프로세스가 종료되는 시점은 대략 400초 후에 **동시에**종료가 된다.  
하지만 Round Robin이 아닌, Nonpreemptive 방식과 같이 하나의 프로세스씩 순차적으로 실행한다면,  
100초 단위로 프로세스가 1개씩 순차적으로 진행되다가 최종 400초가 되는 시점에 마지막 프로세스가 종료된다.  

이는 얼핏 비슷해보이지만 상당히 큰 차이를 갖는다. 400초 동안 프로세스를 4개를 동시에 메모리에 올리고있는 리소스와 1개씩 순차적으로 점유하는 리소스가 다르기 때문이다.


## Multilevel queue  
![](/assets/images/study/dev/2021/os/ch5_highest-priority.jpg)
<figcaption class="caption">Multilevel Queue의 우선순위</figcaption>

멀티레벨 큐는 그림과 같이 우선순위를 갖기 때문에, 하나의 CPU에서 순차적인 처리를 한다면 그림의 순서대로 CPU를 할당받게 될 것이다.

### Multilevel Queue 특징
- Ready queue를 여러 개로 분할
  - foreground(*interactive*)
    - 사람과 interaction이 일어나는 foreground job
  - background(*batch - no human interaction*)
    - batch process와 같은 CPU bounded job
- 각 큐는 독립적인 스케줄링 알고리즘을 가짐
  - foreground - RR
    - interaction이 빈번히 발생하는 프로세스는 언제까지 CPU를 점유해야할지 판단하는 것이 어렵기 떄문에 `Round Robin`방식이 적용
  - background - FCFS
    - CPU bounded job은 큐에 들어온 순서대로 처리하는 것이 유리함
- 큐에 대한 스케줄링이 필요
  - Fixed priority scheduling
    - 위 그림과 같이 우선순위를 고정할 경우 우선순위 잡들이 계속해서 들어오게 된다면, 우선순위 낮은 잡은 영원히 실행되지 못할 수 있다.(`starvation`)
  - Time slice
    - `Starvation`을 방지하기 위하여 각 큐에 CPU time을 적절한 비율로 할당
      - Eg., 80% to foreground in RR, 20% to background in FCFS  

## Multilevel Feedback Queue  
Multilevel Queue와 같이 고정적인 우선순위를 갖게되는 것에 문제가 있는 것을 보완하기 위하여 `Multilevel Feedback Queue`와 같이 여러 줄로 줄을 서며, 서로 간에 이동을 할 수있는 스케줄링 방식이 나왔다.  
![](/assets/images/study/dev/2021/os/ch5_multilevel_feedback_queue.png)

### Multilevel Feedback Queue 특징
- 프로세스가 다른 큐로 이동 가능
- 에이징<sub>aging</sub>을 이와 같은 방식으로 구현 가능
- Multilevel-feedback-queue scheduler를 정의하는 파라미터들
  - Queue 수
  - 각 큐의 scheduling algorithm
  - Process를 상위 큐로 보내는 기준
  - Process를 하위 큐로 내리는 기준
  - 프로세스가 CPU서비스를 받으려 할 때 들어갈 큐를 결정하는 기준

### 일반적인 Multilevel Feedback Queue 운영 방식
- 처음 들어오는 프로세스는 우선순위가 **가장 높은 큐**에 삽입한다
  - 우선순위가 가장 높은 큐는 `RR`을 통해 스케줄링되며 time quantum의 할당시간을 짧게 준다
- 그 다음 프로세스가 들어올 수록 `RR`의 quantum시간을 점점 길게 준다
- 마지막으로 세워진 큐는 `FCFS` 스케줄링을 적용한다

- 그림과 같이 3개의 큐가 존재한다
  - Q0 - time quantum 8 milliseconds
  - Q1 - time quantum 16 milliseconds
  - Q2 - FCFS
- 스케줄링 순서
  - 새로운 잡이 Q0으로 들어감
  - CPU를 할당받은 후 8 milliseconds 동안 수행됨
  - 8ms 동안 끝내지 못하는 경우 Q1으로 내려감
  - Q1에서 줄을 서서 기다린 후 16ms 동안 수행
  - 16ms동안 끝나지 못하는 경우 Q2로 내려감

---

## Multiple-Processor Scheduling
지금까지 다루었던 것은 1개의 CPU를 사용한다고 가정한 방식들을 살펴보았다.  
지금부터 CPU가 복수개 존재할 때 스케줄링을 살펴보자.

- CPU가 여러 개 존재하는 경우 스케줄링은 더욱 복잡해짐
- Homogeneous processor인 경우
  - Queue에 한줄로 세워서 각 프로세서가 알아서 꺼내가게 할 수 있다
  - 반드시 특정 프로세서에서 수행되어야 하는 프로세스가 있는 경우에는 문제가 더욱 복잡해짐
- Load sharing
  - 일부 프로세서에 job이 몰리지 않도록 부하를 적절히 공유하는 메커니즘 필요
  - 별개의 큐를 두는 방법 vs 공동 큐를 사용하는 방법
- Symmetric Multiprocessing(SMP)
  - 각 프로세서가 각각 알아서 스케줄링을 결정 *(프로세스가 모두 대등하게 실행)*
- Asymmetric multiprocessing
  - 하나의 프로세서가 시스템 데이터의 접근과 공유를 책임지고 나머지 프로세서는 거기에 따름 *(하나의 프로세스가 역할을 분담해줌)*


## Real-Time Scheduling
Real-time process는 정해진 시간(dead-line)안에 프로세스가 실행되어야 하는 특징을 갖는다.

- Hard real-time systems
  - Hard real-time task는 정해진 시간안에 반드시 끝내도록 스케줄링되어야 함
- Soft real-time computing
  - Soft real-time task는 일반 프로세스에 비해 높은 priority를 갖도록 해야 함


## Thread Scheduling
Thread : 프로세스에서 CPU 수행 단위

- Local Scheduling
  - User level thread의 경우 사용자 수준의 thread library에 의해 어떤 thread를 스케줄 할지 결정
  - OS가 결정하는 것이 아닌, 사용자 프로세스가 직접 어떤 thread에 CPU를 할당할지 결정함
- Global Scheduling
  - Kernel level thread의 경우 일반 프로세스와 마찬가지로 커널의 단기 스케줄러가 어떤 thread를 스케줄할지 결정
  - 운영체제가 스케줄 알고리즘에 의해 CPU를 할당

## Algorithm Evaluation
지금까지 다양한 CPU Schedling 알고리즘을 살펴보았고, 각 알고리즘의 특징을 살펴보았다.  
이제 그 알고리즘의 평가하는 방법에 대해서 살펴보도록 하자.

- Queueing models
  - 확률 분포로 주어지는 `arrival rate`와 `service rate`등을 통해 각종 `performance index` 값을 계산
    - CPU 잡을 수행한 도착률 & 처리률을 계산하여 throughput을 계산한다
    - 이론적인 방식
- 구현 & 성능 측정(Implementation & Measurement)
  - 실제 시스템에 알고리즘을 구현하여 실제 작업(workload)에 대한 성능을 측정 비교
- 모의 실험(Simulation)
  - 알고리즘을 모의 프로그램으로 작성 후 trace를 입력으로 결과 비교
  - `trace` : 실제 프로그램에 들어갈 input data


---

*Reference*

- Operating System Concepts (Paperback / 9th Ed.) Books
- http://www.kocw.net/home/search/kemView.do?kemId=1046323