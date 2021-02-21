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

---

*Reference*

- Operating System Concepts (Paperback / 9th Ed.) Books
- http://www.kocw.net/home/search/kemView.do?kemId=1046323