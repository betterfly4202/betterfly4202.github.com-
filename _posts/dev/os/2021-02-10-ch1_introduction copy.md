---
title: "[Operating System] Ch1. 운영체제의 기본"
categories: "OS"
tags:
  - Operating System
  - System Structure
  - Program Execution
---

# 운영체제란 무엇인가?
## 운영체제의 정의
![](/assets/images/study/dev/2021/os/ch1_os_layer.png)

그림처럼 하드웨어와 소프트웨어 중간에 위치하여, 모든 소프투웨어와 하드웨어으를 연결해주는 **소프트웨어 계층**을 말한다.  
좁은의미로 운영체제는 운영체제의 핵심으로 메모리에 상주하고 있는 커널<sub>kernel</sub>을 말하며,  
넓은의미의 운영체제는 커널뿐만 아니라, 메모리에 포함되지 않는 각종 주변 시스템 유틸리티를 포함한다.

다시 정리하면, 운영체제는 **하드웨어 자원을 효율적으로 관리하는 목적**으로 설계된 소프트웨어를 말한다.

## 운영체제의 분류

운영체제는 다음의 3가지로 분류될 수 있다.

- 동시 작업 가능 여부
- 사용자의 수
- 처리 방식

### 동시작업 가능 여부
- Single tasking : MS-DOS와 같이 한 명령의 수행을 종료 후 다른 명령을 실행하는 것을 말한다.  
엘레베이터, TV와 같은 단순한 단일 명령만 수행하는 경우를 말한다.  
- Multi tasking : UNIX, Windows와 같은 소프트웨어하에서 동시에 다양한 명령을 실행하는 것을 말한다. (실제로 **동시**보다는 아주 빠른속도로 `context switching`이 일어난다)

### 사용자의 수
컴퓨터 한 대를 동시에 접근 가능한 경우
- Single user: MS-DOS, Windows 등
- Multi user : UNIX, NT Server(하나의 서버 장비에 여러 계정을 생성하여 동시에 접근이 가능)

### 처리 방식
- 일괄처리(batch processing) : 작업 요청의 일정량을 모아서 처리하는 방식을 말한다. (작업이 완료 종료시점까지 대기해야 함)
- 시분할(time sharing) : 여러 작업을 수행시 컴퓨터 처리 능력을 일정 시간 단위로 분할 `interactive`한 방식 -> 위에 설명한 동시 작업의 **multi tasking**에 해당
- 실시간 (realtime OS) : 정해진 시간안에 실행한 잡이 반드시 종료되는 것이 보장하는 시스템

## 컴퓨터 시스템 구조
### 시스템의 기본 구조
- CPU(중앙 처리 장치)
  - 인간의 두뇌에 해당하는 것으로, 프로그램 명령어와 데이터를 읽어 처리하고 수행 순서를 제어하는 핵심적인 영역
- Memory(주 기억장치)
  - CPU에게 작업을 할당해주는 기억 장치
- Disk(보조 기억 장치)
  - 영구적으로 보관되는 파일을 저장해주는 공간, 주기억장치(ROM, RAM)에 비해 속도는 느리지만 영구적으로 데이터를 보관해준다.
- I/O Device
  - 키보드, 모니터, 스피커 등 컴퓨터에 연결하는 외부 입력 장치를 말한다.
  - I/O device를 통해 입/출력을 요청시 CPU와 Device Controller간의 interrupt를 통해 처리한다


### CPU Scheduling
앞서 살펴보았던 운영체제를 통해 어떻게 컴퓨팅 시스템을 효율적으로 사용할 것이가는, 다시 말해 CPU를 어떻게 효율적으로 사용할 것인가로 귀결된다.  
이처럼 컴퓨터의 모든 행동을 관장하는 'CPU에게 어떻게 일을 시킬 것인가' 하는 부분이 가장 핵심이다.  
다음 그림을 통해 어떻게 CPU가 일을 할당받고 스케줄링 하는지 살펴보자.
![](/assets/images/study/dev/2021/os/ch1_cpu_scheduling.jpeg)

CPU는 메모리에 바라보고 있으며, 특별한 요청이 없다면 메모리에 쌓인 태스크들을 순차적으로 실행한다.  
다시 말해, CPU는 메모리에 쌓인 instruction을 매 클럭마다 수행하는 것이다.  
하지만 I/O 디바이스를 통해서 요청이 발생하는 경우 I/O 디바이스는 `interrupt`를 발생시켜, CPU에게 작업을 우선적으로 할 것을 요청한다.

위 그림을 조금 더 구체화해보자.

### System Structure
![](/assets/images/study/dev/2021/os/ch1_system_structure.png)

세분화하니까 CPU-Memory-I/O Deivce 사이에 다양한 장치들이 추가되었다.  
그러면 하나씩 각 역할을 살펴보자.

#### mode bit
CPU에서 실행 중인 프로그램이 OS인지, User Program인지를 구분해주는 장치 (사용자 프로그램이 잘못된 수행으로 다른 프로그램 및 OS에 영향을 주지 않도록 하기 위한 보조 장치)
- mode bit은 `0` 또는 `1` 로 구분된다.
  - 0 : 모니터 모드 (커널, 시스템 모드) -> OS 코드 수행
  - 1 : 사용자 모드, 사용자 프로그램 수행 (OS 접근이 제한됨)
- 보안에 위배되는 명령은 모니터 모드(mode bit : 0)에서만 수행이 가능함
- `Interrupt` 혹은 `Exception` 발생시 **mode bit = 0** 으로 바뀜
- 사용자 프로그램이 수행될때는 **mode bit = 1**

#### timer
특정 프로그램이 CPU를 독점하는 것을 방지하기 위한 장치(정해진 시간이 흐른뒤 운영체제에게 제어권이 넘어가도록 `interrupt`를 발생시킴)
- 매 클럭 1씩 감소
- 타이머 값 : 0 -> interrupt 발생
- timer의 활용
  - 프로그램의 CPU 독점 방지
  - time sharing
  - 프로그램에서 현재 시간을 계산

#### Device Controller
I/O 장치를 관리하는 일종의 작은 CPU 역할  
I/O 디바이스의 입력이 CPU에 전달되기까지의 과정을 다음 그림을 통해서 살펴보자
![](/assets/images/study/dev/2021/os/ch1_device_controller.jpeg)

디바이스부터 CPU에 전달되는 순서는 다음과 같다.

1. 키보드 또는 모니터 등 I/O 디바이스에서 데이터를 입력
2. device buffer에 데이터 저장
3. device controller는 버퍼에 일정 데이터가 쌓인 것을 확인 후 **DMA Controller**에게 수행을 요청함
4. DMA Controller는 device controller의 요청을 받은 후 I/O buffer에 쌓인 데이터를 메모리에 복사해줌
5. 메모리에 복사 후 DMA Controller는 CPU에 `interrupt`를 발생시킴
6. interrupt를 받은 CPU는 메모리의 해당 영역(I/O 디바이스의 입력)을 수행

> **DMA Controller의 역할**  
CPU가 메모리영역부터 I/O 입/출력까지 모든 작업을 수행할 경우 I/O 장치가 많아질 수록 모든 입력에 대해 interrupt가 발생시키며 효율이 떨어지게 된다.  
이런 경우 각 I/O의 입력을 buffer에 담아두고 완료된 시점에 CPU에게 알려주어서(interrupt) CPU의 interrupt 빈도를 줄이며 효율을 증가시켜준다.  
이처럼 I/O buffer의 데이터를 memory까지 운반해주는 것이 DMA Controller의 역할이다.

### 입출력(I/O)의 수행
- 모든 입출력 명령은 특권 명령이다.
  - 특권 명령 : 사용자 프로그램이 직접 명령 불가, 운영체제를 통해서만 명령<sub>System Call</sub> 가능
  - System Call : 사용자 프로그램이 직접 제어할 수 없어 운영체제에 요청하는 행위

#### Software interrupt 처리 과정
1. `trap`<sub>software interrupt</sub>을 사용하여 interrupt vector의 특정 위치로 이동
2. 제어권이 interrupt vector가 가리키는 interrupt service routine으로 이동
3. 올바른 I/O요청인지 확인 후 I/O 수행
4. I/O완료 후 제어권을 System call의 다음 명령으로 옮김

![](/assets/images/study/dev/2021/os/ch1_software-interrupt.png)

우리가 프로그래밍을 할때를 떠올리며 그림을 통해 **Software interrupt(trap)**가 발생하는 과정을 살펴보자.  

예를들어, 우리가 Java 프로그램을 개발하여 실행했다고 가정해보자.  
그러면 해당 자바 프로세스가 실행되며, 이 프로세스는 컴퓨터 Memory 중 JVM안에서 관리될 것이다.  
그리고 우리가 만들어 놓은 모든 프로그램 함수 및 변수들은 JVM안에서 실행되고 관리된다.  
그런데 함수 로직 중 `파일I/O` 또는 `네트워크I/O`등 I/O가 필요한 경우 System Call이 일어나게 되는 것이다.

1. I/O요청 시 CPU에 interrupt 발생
2. mode bit = 1 (사용자 프로그램 실행) 에서 mode bit = 0 으로 변경 (OS 기능을 사용하기 위함 -> I/O)
3. CPU제어권이 OS에게 넘어감
4. OS는 device controller에게 I/O요청
5. Device Controller는 DMA를 통해서 자신의 버퍼에 있는 데이터를 memory로 이관(복제)

![](/assets/images/study/dev/2021/os/ch1_transition_user2kernel.png)
---

### 인터럽트(Interrupt)
인터럽트 당한 시점의 레지스터와 프로그램 카운터<sub>program counter</sub>를 저장 후 CPU의 제어를 `Interrupt Service Routine`에 넘겨준다.
- Interrupt 
  - 하드웨어 영역의 인터럽트
  - Timer, I/O Controller 등에서 일으킴
- Trap
  - 소프트웨어 영역의 인터럽트
  - Exception(프로그램 오류), System Call(프로그램이 커널 함수를 호출)하는 과정에서 일으킴

> **용어 정리**  
> **Interrupt Vector** : 어떤 Intrrupt를 처리할지에 대한 메모리 주소, 해당 intrrupt의 처리 루틴 주소를 가지고 있음  
> **Interrupt Service Routine(Interrupt Handler)** : 해당 inrrupt를 처리하는 kernel 함수

-> 즉 CPU는 매 **interrupt 마다 처리할 일**<sub>interrupt vector</sub>을 **kernel내 정의된 함수**<sub>interrupt service routine</sub>로 처리한다.

---

*Reference*

- Operating System Concepts (Paperback / 9th Ed.) Books
- http://www.kocw.net/home/search/kemView.do?kemId=1046323