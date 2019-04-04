---
title: "Process와 Thread"
categories: "JAVA"
tags:
  - Process
  - Thread
---

## Process와 Thread이야기
Process 와 Thread 에 대한 개념은 흔히 언급되는 내용이다.
언뜻 비슷해보이지만, 큰 차이가 있는 이 두 개념을 혼용하지 않기 위하여.
그리고 기본 개념을 다지기 위하여 기록한다.

일반적으로 프로세스와 스레드에 대하여 쉽게 정의하면 다음과 같다.

>*Process* : 운영체제로부터 자원을 할당받는 작업의 단위

>*Thread* : 프로세스가 할당받은 자원을 이용하는 실행 단위


### Process
말장난 같은 이야기지만, 일반적으로 프로세스는 우리가 흔히 운영체제(윈도우 또는 리눅스와 같은)에서 프로그램을 실행하는 행위에 동작한다.

프로그램 구동시 운영체제에서는 프로세서를 할당하여 운영에 필요한 주소 공간, 메모리 등의 자원을 할당받는다. 프로그램은 하나의 프로세스에 의해 실행이 될 수도 있으며, 
어떤 작업을 하나 이상의 프로세스에서 병렬로 처리하는 것을 '멀티 프로세싱(Multi Processing)' 이라고 한다.

또한 과거 MS-DOS시절과 달리, 최근 사용되는 운영체제는 여러개의 프로세스를 동시에 실행할 수 있는데, 이것을 '멀티 태스킹(Multi Tasking)' 이라고 한다.

프로세스는 각각의 독립된 메모리 영역에서 작동한다.
 
![Process_Menory](/assets/images/study/dev/2018/8_memory_organization.jpg)


이러한 메모리 구조를 사용하는 프로세스는 CPU에서 적재된 프로세스의 로테이션을 돌며 '멀티 프로세싱'하는 과정에서 문제점을 보인다.

A라는 프로세스가 동작 후 대기중이던 B 프로세스가 실행될 때, A프로세스의 상태(context)는 보관되며 B프로세스의 상태(context)로 교환하는 Context Switching 작업이 일어 난다.

이 과정에서 각각 독립된 메모리 영역에 존재하는 프로세스들은 캐쉬 메모리 초기화 등의 무거운 작업을 반복하며 *Overhead* 를 발생시킬 수 있다.


### Thread
스레드는 프로세스 내에서 실행되는 흐름의 단위이다. 다시 말해, 하나의 프로세서에서 진행되는 각각의 일의 단위이다.

기본적으로 프로세스 당 최소 1개의 스레드가 존재하며, 그것을 *Main Thread* 라고 부른다. (프로세스 상에 2개 이상의 스레드가 존재한달 경우 멀티 스레드(Multi Thread)라고함.)

스레드는 프로세스상에 실행되기 때문에 하나의 프로세스에 적재되어 Stack 영역만 따로 할당받으며, Code, Data, Heap 영역은 공유되어 사용된다.

![Thread_Menory](/assets/images/study/dev/2018/8_thread_container.png)

메모리를 공유하는 스레드는 *전역변수*를 통해서 데이터를 공유하고, 시스템의 자원 소모를 줄일 수 있어 효율을 높일 수 있다.

특히 프로세스의 경우 Context Switching이라는 높은 비용을 감수해야 하지만, 스레드의 경우 이런 큰 처리 비용은 고려하지 않아도 된다.

하지만, 멀티 스레드를 사용할 경우 공유된 데이터간의 충돌과 까다로운 디버깅, Dead-Lock 등을 유의하여 사용해야 한다.


#### 간단한 스레드의 예시

~~~java
public class ThreadTest {

    public static void main(String [] args) throws InterruptedException{
        final Thread separateThread = new Thread(new ThreadPrinter());
        separateThread.start();
        for(int i=0; i<5; i++){
            System.out.println("Main Thread..." + Thread.currentThread().getName());

            Thread.sleep(1000);
        }

    }
}



public class ThreadPrinter implements Runnable{
    @Override
    public void run(){
        for(int i=0; i<5; i++){
            System.out.println("new Thread..." + Thread.currentThread().getName());
            try{
                Thread.sleep(1000);
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }
    }
}

~~~
![thread](/assets/images/study/dev/2018/8_thread_test.png)

---

#### Dead Lock
Dead Lock에 대해서는 유명한 '철학자의 식사문제' 이야기가 있다.

![dining_philosopher](/assets/images/study/dev/2018/8_An_illustration_of_the_dining_philosophers_problem.png)

##### [다섯 명의 철학자와 포크]

- 다섯 명의 철학자가 원탁에 앉아 있고, 각자의 앞에는 스파게티가 있고 양옆에 포크가 하나 씩 있다. 

- 그리고 각각의 철학자는 다른 철학자에게 말을 할 수 없다. 

- 철학자가 스파게티를 먹기 위해서는 양 옆의 포크 두개를 동시에 들고 있어야 한다. 

- 이때 각각의 철학자가 왼쪽의 포크를 들고, 그 후 오른쪽의 포크를 들어야 하는 규칙이 있다고 할 때, 

- 다섯 철학자가 동시에 왼쪽의 포크를 든다면, 오른쪽에 있는 포크는 손에 쥘 수 없으므로(옆의 철학자가 쥐고 있이므로) 무한정 기다려야 하는 *교착 상태(Dead Lock)* 에 빠지게 될 수 있다.

- 또한 어떤 경우에는 동시에 포크 양쪽을 집을 수 없어 식사를 하지 못하는 기아 상태가 발생할 수도 있고, 몇몇 철학자가 다른 철학자보다 식사를 적게 하는 경우가 발생하기도 한다.

---

[참고]
- https://brunch.co.kr/@kd4/3
- https://magi82.github.io/process-thread/
- http://icednut.github.io/2016/08/06/20160806-about_deadlock/

