---
title: "[Operating System] Ch6. Process Synchronization"
categories: "OS"
tags:
  - Operating System
  - Process Synchronization
---

# Process Synchronization
![](/assets/images/study/dev/2021/os/ch6_data_access.png)
일반적으로 데이터의 접근은 그림과 같이,
1. 데이터를 저장하는 공간이 있다.
2. 연산할 데이터를 선택한다.
3. 연산 처리자는 해당 데이터를 연산한다.
4. 연산된 결과를 반환하여 원래의 저장소에 전달한다.

일반적으로 데이터를 읽기만하거나, 읽은 후 위 절차에 따라 순서대로 진행한다면 문제가 없다.  
하지만 이 저장과 연산이라는 흐름속에서 어떤 데이터를 먼저 읽었는지, 연산했는지에 따라서 최종 결과가 달라질 수 있는데, 이러한 문제를 `Synchronization`이라고 한다.

## Race Condition
**Process synchronization**의 문제를 대표하는 것이 바로 `race condition`이다.  
![](/assets/images/study/dev/2021/os/ch6_race-condition.png)

그림과같이 저장된 `count`라는 데이터를 한쪽에선 증가(count++)시키고, 한쪽에서는 감소(count--)하는 연산자가 있다고 하자.  
두 연산자의 순서가 어떻게 실행했는지에 상관없이 양쪽의 연산을 순차적으로 진행하면 (증가-> 감소 or 감소 -> 증가) 결과는 **연산을 하기 전의 count 값**이 된다.  
*하지만 만약에, 증가 연산을 하는 도중에, 감소 연산을 하게 되면 어떻게 될까?*  
연산 순서에 따라 뒤에 감소 연산의 결과가 저장되어, 결과적으로 **감소(count--)된 결과만 저장**되게 될 것이다. 그 반대의 경우도 마찬가지이다.

이처럼 여러 연산자들이 하나의 저장된 데이터를 동시에 접근할때 발생하는 문제를 `race condition`이라고 한다.

정리하자면, 
- 공유 데이터(shared data)의 동시 접근(con-current access)은 데이터의 불일치 문제(inconsistency)를 발생 시킨다.
- 일관성(consistency) 유지를 위하여 협력 프로세스(cooperating process)간의 실행 순서(orderly execution)를 정해주는 메커니즘이 필요하다.

- Race Condition 발생 요인
  - 여러 프로세스들이 동시에 공유 데이터를 접근하는 상황
  - 데이터의 최종 연산 결과는 마지막에 그 데이터를 다룬 프로세스에 따라 달라진다.

**race condition을 막기 위해서는 concurrent process는 `동기화(synchronize)`되어야 한다.**

## Critical Section
- n개의 프로세스가 공유 데이터를 동시에 사용하기를 원하는 경우
- 각 프로세스의 code segment에는 공유 데이터를 접근하는 코드인 `critical section`이 존재
- 하나의 프로세스가 *critical section*에 있을때 다른 모든 프로세스는 critical section에 접근하지 못해야 한다.

![](/assets/images/study/dev/2021/os/ch6_critical_section.png)
`Critical section`은 그림처럼, 데이터의 영역이 아니라 데이터를 접근하고 있는 **소스코드의 영역**을 말한다.

### Critical section을 해결하기 위한 전략
```c++
  do {
    entry section
    critical section
    exit section
    remainder section
  } while(1)
```

critical section을 해결하기 위해서 위 구조처럼 critical section을 진입하기 전에 lock을 걸어서 다른 프로세스의 진입을 차단하고(entry section),  
critical section의 작업이 완료된 후 lock을 해제하여(exit section) 다음 프로세스의 접근을 허용한다.

###  프로그램적 해결법의 충족 조건
#### Mutual Exclusion(Mutex)
프로세스 P<sub>i</sub>가 critical section 부분을 수행 중이면 다른 모든 프로세스들은 그들의 critical section에 들어가면 안된다.

Mutual Exclusion추가 설명
- 0또는 1의 값을 가지는 이진 세마포어와 유사
  - Critical Section(임계구역)을 가진 스레드들의 실행 시간을 서로 겹치지 않게 단독으로 실행하게 하는 기술
- 프로세스들의 공유 리소스에 대한 접근을 조율하기 위해 `Locking`과 `Unlocking`을 사용
- 뮤텍스 객체를 두 스레드가 동시에 사용할 수 없다


#### Progress
아무도 critical section에 있지 않은 상태에서 critical section에 들어가고자 하는 프로세스가 있으면 critical section에 들어가게 해주어야 한다.

#### Bounded Waiting
프로세스가 critical section에 들어가려고 요청한 후부터 그 요청이 허용될때까지 다른 프로세스들이 critical section에 들어가는 횟수에 한계가 있어야 한다.

### Critical Section을 해결하기 위한 알고리즘
위 해결법을 참고하여, 어떻게 소프트웨어 적으로 프로세스의 lock을 컨트롤할 수 있을지 다음의 알고리즘 전략들을 통해서 살펴보자.

#### Algorithm 1

![](/assets/images/study/dev/2021/os/ch6_algorithm_1.png)
첫번째 알고리즘은 그림과 같이 프로세스2개가 있다고 가정해보자.  

P<sub>0</sub> 프로세스 기준으로 살펴보면, 
**while(turn !=0)** 0이 아닌 경우 계속 while문을 진입하지못하고 계속 turn의 값을 체크 후 `turn==0`이 되는 순간 while문에 진입하여 *critical section*을 수행 후 turn의 상태를 변경해준다. 이 순간 P<sub>1</sub>에서는 `turn==1`이 된 것을 확인하여 자신의 *critical section*을 수행할 수 있게 된다.

쉽게 말해서 순서를 차례대로 정의하여 한번씩 순차적으로 실행하도록 유도하는 알고리즘이다.

따라서 이 코드는 두 프로세스가 동시에 크리티컬 섹션에 진입하지 않기 때문에 `Mutual Exclusion`에 만족한다.  
하지만 문제가 있는데, 이 방식은 크리티컬 섹션에 반드시 교대로 진입할 수 있도록 되어 있다. 
이 말은 하나의 프로세스에서 자신의 크리티컬 섹션에 더 빈번하게 접근하여 수행주기를 높이고 싶지만, 반드시 균일하게 한쪽의 프로세스가 완료되어야만 다음 프로세스로 CPU를 넘겨주기 때문에 `Progress`의 조건에는 만족하지 않는다.

#### Algorithm 2
- Synchronization variables
  - 2개의 boolean 변수(초기값 모두 `false`)
  - P<sub>i</sub> 프로세스 먼저 크리티컬 섹션에 진입 준비 (**flag[i] == true**)
- Process P<sub>i</sub>

```c++
  // P[i] 프로세스 먼저 크리티컬 섹션 진입
  do{
    flag[i] == true; // 'i' 프로세스의 크리티컬 섹션 진입 선언
    while(flag[j]);  // 'j' 프로세스의 상태 확인
    <<critical section>>
    flag[i] = false; // 'i' 프로세스 진행 종료
    <<remainder section>>
  }while(true);
```

이 알고리즘은 얼핏보면, 교대로 자신의 순서를 선언하고 진행하기 때문에 문제가 없는것처럼 보인다.  
하지만 만약에, 
- P<sub>i</sub> 프로세스에서 **flag[i] == true;** 로 표기한 직후에
- CPU가 P<sub>j</sub> 프로세스에게 넘어가서 **flag[j] == true;**
위 상태가 된다면 어떨까?  
결과적으로 *flag[i] == true && flag[j] == true* 의 상태가 되는 것이다. 그 다음 while문을 실행하려고 보니 서로가 깃발을 들고 있기 때문에 서로의 프로세스가 종료되기를 기다리기만 하게 된다. 
![철학자와_식사를_통한_데드락](https://better-dev.netlify.app/java/2018/08/12/proces_thread/)


#### Algorithm 3(Peterson's Algorithm)
![](/assets/images/study/dev/2021/os/ch6_peterson.png)

이 알고리즘의 차이점은 **Algorithm 2**에서 진화하여, 상대 프로세스의 차례(`flag[j]`)인지 확인하는 것과, 자신의 턴이 맞는지(`turn=j`)를 동시에 확인한다.  
두가지 중 하나라도 만족하지 않으면 자신의 크리티컬섹션에 진입 후 상대에게 차례를 넘기게 되는 것이다.

위 알고리즘은 앞소 알고리즘1,2의 문제점을 보완하여 `Mutual Exclusion`, `Progress`, `Bounded Waiting`의 문제를 모두 해결해준다.  

##### Busy Waiting(spin lock)
피터슨의 알고리즘은 모든 문제를 해결한것처럼 보이지만, 이 역시 완벽하진 않다.
바로 **Busy Waiting(=spin lock)**의 문제를 일으킨다는 것인데,  
어느 한 쪽이 크리티컬섹션을 수행중에 CPU가 넘어가서 다른 프로세스의 while문을 체크하게되는데 이 while문은 두 조건을 만족하지 못하니까 계속 while문을 수행할 것이고, 다시 CPU가 크리티컬섹션을 수행중인 곳에 넘어가서 이어서 다음 부분을 수행하고 다시 while문은 조건이 맞는지를 검증하기위하여 CPU와 Memory를 지속적으로 사용하면서 대기를 하게 된다.

이렇게 불필요하게 CPU와 Memory를 낭비하며 리소스를 점유하는 현상을 **Busy Wating(=Spin lock)**이라고 한다.

### Synchronization Hardware
크리티컬섹션에 대한 문제를 소프트웨어로 해결하려고 보면, 위처럼 다양한 경우의 수를 따라 복잡한 알고리즘을 고민해야 한다.  
하지만 이를 하드웨어에서 접근하면 비교적 단순하게 해결이 가능하다.  

동기화 문제는 하나의 프로세스가에서 각 코드라인을 실행 중 불특정한 구간에서 CPU를 빼앗길 수 있기 때문에 위와같은 복잡한 알고리즘을 통해서 크리티컬섹션을 확보하기 위한 고민이 담긴 것이다.  
이것을 하드웨어적으로 접근하여, 앞서 프로세스의 코드를 수행하는데 하나의 instruction으로 처리되도록 접근하면되는데 이것을 `Test_and_set`이라는 instruction이라고 한다.  

지금 문제는 하나의 데이터를 읽기(read)와 쓰기(write)를 하나의 instruction으로 처리할 수 없기 때문에 문제인 것인데, 만약 데이터를 읽으면서 동시에 쓰는 것을 하나의 instruction으로 수행하는 것을 보장한다면 (instruction 하나가 수행하는 도중에 CPU를 빼앗기진 않기 때문에, interrupt는 instruction이 끝났을 때 적용된다) 간단하게 lock을 조절할 수가 있다.

#### TEST_AND_SET
> 하드웨어적으로 Test & Set을 **atomic**하게 수행할 수 있도록 지원

![](/assets/images/study/dev/2021/os/ch6_test_and_set.png)
앞서 설명한 **읽고-쓰는 것**을 하드웨어적으로 해결하기 위한 instruction이 `TEST_AND_SET`이다.  
그림처럼 변수 `a`를 읽고<sub>read</sub> `a = true`로 바꿔주는<sub>write</sub> 행위를 하나의 instruction으로 처리하는 것이다.  

### Semaphores
프로세스간의 시그널(신호, Signal)을 주고받기 위해 사용되는 **정수 값**으로, 리소스의 상태를 나타내는 카운터로 세마포어는 다음 세가지 원자적인<sub>atomic</sub> 연산만을 지원한다.
- initialize : 세마포어 초기화. (음이 아닌 정수값으로 초기화)
- decrement : 프로세스를 블록시킴(Lock)
- increment : 블록되었던 프로세스를 깨워줌(Unlock). 이 세마포어를 `카운팅 세마포어` 또는 `범용 세마포어`라고 한다

출처: https://about-myeong.tistory.com/34 [명찌의 포스트잇]

#### 추상자료형
  - Object와 Operation으로 구성된다.
  - **정수 추상자료형** 이라고 하면, 정수<sub>Object</sub> 숫자들이 있고들이 있고 그 숫자에 대해서 정의된 연산<sub>Operation</sub>덧셈,뺼셈 등이 정의되어 있는 것을 말한다.
  - 쉽게 말해서 프로그래밍에서 interface의 개념으로 보면되는데, 사용자들은 클래스에 어떤 interface가 있는지만 알면 내부적으로 어떤 구현 방식인지 관여하지 않고 사용자 목적에 맞게 사용하는 것을 말한다.

#### Semaphore 'S'
- 세마포어도 일종의 **추상자료형**
- Semaphore 변수`S`는 정수값(integer variable)을 가질 수 있다. -> `Object`
- 그리고 Semaphore 'S'의 연산을 가진다. -> `Operation`
  - 그림의 두 연산은 `atomic`하게 수행된다는 것을 가정함
![](/assets/images/study/dev/2021/os/ch6_Semaphores_1.png)
  - P(S) (P연산)
    - 공유 데이터를 획득하는 과정
    ```C++
      while(S <=0){
        do no-op;
      }
      S--;
    ```
  - V(S) (V연산)
    - 데이터를 획득 후 반납하는 과정
    ```c++
      S++;
    ```
  - P,V 두 연산은 atomic연산에 의해서만 접근이 가능하다



---

### Bounded-Buffer Problem
> **Buffer**  
임시로 데이터를 저장하는 공간

![](/assets/images/study/dev/2021/os/ch6_bounded-buffer-problem.png)

그림과 같이 메모리 안에 버퍼가 존재하고 생산자(Producer)와 소비자(Consumer) 관계가 있다고 보자.  
생산자는 **버퍼가 비어있다면** 데이터를 생산하는 주체이고, 소비자는 **버퍼안에 데이터가 존재하면** 가져다가 사용하는 주체이다.  

여기에 몇가지 Synchronization 문제를 예상해 볼 수 있는데
- 2개의 생산자<sub>Producer</sub>가 하나의 버퍼에 동시에 접근하려는 경우
- 마찬가지로 2개의 소비자<sub>Consumer</sub>가 하나의 공유된 자원(버퍼)에 접근하려는 경우

위 두 경우 먼저 도착한 생산자가 자신이 작업을 수행하기 전에 공유 자원에 `Lock`을 걸어서 다른 생산자가 접근하지 못하도록 한다.

또 다른 문제는 **버퍼가 유한**하기 떄문에 발생하는 문제가 있다.

**생산자 관점**  
- 사용할 수 있는 모든 버퍼에 데이터를 생산했다.
- 모든 버퍼에 데이터가 쓰였는데, 또 다른 생산자가 진입하여 데이터를 생산하려고 한다.
- 사용할 수 있는 자원(데이터가 비어있는 버퍼)이 존재하지 않음
- 소비자가 버퍼를 사용하여 가용할 버퍼가 발생할때까지 대기

**소비자 관점**
- 버퍼의 모든 데이터를 소비자가 소진했다.
- 버퍼에 사용할 데이터가 없는데, 다시 소비자가 진입하여 데이터를 소진하려고 한다.
- 사용할 수 있는 자원(데이터가 존재하는 버퍼)가 존재하지 않음
- 생산자가 버퍼를 생산해줄때까지 대기

#### Bounded-Buffer Problem의 동기화 변수
- mutual exclusion : 공유 자원의 사용에 대해 Lock을 걸어주고, 풀어주는 변수
- resource count : 생산자/소비자 관점에서 사용할 수 있는 자원(buffer)의 수

**synchronization variables**  
- semaphore full = 0; 
- empty = n; 
- mutex = 1;

**Producer process**
```c++
  do{
    ...
    produce an item in x
    ...

    P(empty);  // 내용이 비어있는 버퍼 확인
    P(mutext); // 버퍼에 Lock을 걸기
    ...
    add x to buffer
    V(mutex);  // 작업 완료 후 Lock 풀기
    V(full);   // 작업이 완료되어 버퍼의 내용을 채워줌(소비자 입장의 자원 알림)
  }while(1);
```

**Consumer process**
```c++
  do{
    P(full);  // 내용이 채워있는 버퍼를 확인
    P(mutex); // 버퍼를 사용하기 위해 Lock 걸기
    ...
    remove an item from buffer to y
    ...
    V(mutex); // 작업 완료 후 Lock 풀기
    V(empty); // 비어있는 버퍼의 내용을 1 증가시켜줌(생산자 입장의 자원 알림)
    ...
    consume the item in y
  }wihle(1);
```

#### Readers-Writers Problem
- 한 프로세스가 DB에 write 중일 때 다른 프로세스가 접근하면 안됨
- read는 동시에 여럿이 접근이 가능함
- solution
  - Writer가 DB에 접근 허가를 아직 얻지 못한 상태에서는 모든 대기중인 Reader들을 다 DB에 접근하게 해준다.
  - Writer는 대기 중인 Reader가 하나도 없을 때 DB접근이 허용된다.
  - 일단 Writer가 DB에 접근 중이면 Reader들은 접근이 금지된다.
  - Writer가 DB에서 빠져나가야만 Reader의 접근이 허용된다.


**Shared data**
- DB 자체
- readcount : 현재 접근중인 Reader의 수

**Synchronization variables**
- semaphore mutext = 1;
- db = 1;

**Writer**
```C++
  P(db); // Lock걸기
    ...
    writing DB is performed
    ...
  V(db);  // Lock풀기
```
위 코드는 간단하다.  
Writer가 실행되면 다음 Writer와 충돌하지 않도록 Lock을 걸어 진입을 제한하고 자신의 writing 작업을 마친 후 Lock을 풀어준다.

**Reader**
```C++
  P(mutex);
  readcount++;
  if(readcount == 1){
    P(db) // block writer, readers follow
  }
  V(mutex);
    ...
    reading DB is performed
    ...
  p(mutex);
  readcount--;
  if(readcount == 0){
    V(db); // enable writer
  }
  V(mutex);
```
Reader는 상대적으로 복잡해 보이지만, 역시 간단 하다.  
DB의 데이터를 읽을때 마찬가지로 Lock을 걸어준다. 이 Lock은 Read 중 Writer의 작업을 제한하지만 다른 Reader들의 접근은 허용하여 읽기 작업은 추가로 수행할 수 있다.  
그리고 마지막의 Reader작업까지 마친 후 Lock을 풀어주어(readcount--) 다음 Writer작업을 허용하게 해준다. 

**Starvation 발생 가능**  
위와 같이 Reader를 우선순위로 작업시, Reader작업이 끊임없이 들어온다면 Writer는 무한히 대기를 하기 때문에 `Starvation`문제가 발생한다.

#### Dining-Philosophers Problem
![](/assets/images/study/dev/2021/os/ch6_dining-philosophers.png)
이 문제는 앞서 언급했던 ![철학자와_식사를_통한_데드락](https://better-dev.netlify.app/java/2018/08/12/proces_thread/) 문제인데 부가적인 설명을 덫붙인다.

- 5명의 철학자가 원탁에 둘러 앉아 있다.
- 이들은 모여서 생가을 하거나(*think()*), 밥을 먹는다(*eat()*)
- 식사시 자신의 왼쪽, 오른쪽의 젓가락을 순서대로 하나씩 들어야 식사가 가능하다.
- 자신의 식사를 마친 후 들었떤 왼쪽, 오른쪽 젓가락을 순서대로 자리에 다시 놓는다.

이러한 흐름대로라면 그림의 1번 철학자가 식사를 시작하면 옆에 있는 2번 철학자는 1번 철학자가 식사 마치는것을 기다려야 한다.

```c++
  do{
    P(chopstick[i]);
    P(chopstick[(i+1)%5]);
      ...
      eat();
    V(chopstick[i]);
    V(chopstick[i+1]%5);
      ...
      think();
      ...
  }while(1);
```
**DeadLock 가능성**
- 모든 철학자가 동시에 식사를 위해 왼쪽 젓가락을 집어버리는 경우

**해결책**
- 4명(짝수)의 철학자만이 테이블에 동시에 앉을 수 있도록 한다
- 양쪽의 젓가락을 모두 잡을 수 있을떄에만(젓가락 상태 확인) 젓가락을 잡을 수 있게 한다
- 비대칭
  - 짝수(혹은 홀수) 철학자는 왼쪽(혹은 오른쪽)의 젓가락을 잡도록 한다


### Monitor
Semaphore의 한계
- 코딩하기 어렵다
- 정확성(correctness)의 입증이 어렵다
- 자발적 협력(voluntary cooperation)이 필요하다
- 한번의 실수가 모든 시스템에 치명적으로 영향을 준다

**Mutual exclusion이 깨지는 상황**
```c++
  V(mutex)
  // Critical Section
  P(mutex)
```

**Dead Lock이 발생하는 상황**
```c++
  P(mutex)
  // Critical Section
  P(mutex)
```

- 동시 수행중인 프로세스 사이에서 abstract data type의 안전한 공유를 보장하기 위한 high-level synchronization construct

![](/assets/images/study/dev/2021/os/ch6_semaphore-monitor.png)
- 모니터 안에 공유 데이터와 접근 코드를 정의함
- 모니터를 통해 데이터와 코드를 통제하여 액티브되는 프로세스가 1개만 실행될 수 있도록 제어해줌

![](https://www.youtube.com/watch?v=PQ5aK5wLCQE)
유튜브 영상은 `세마포어`로 설명을 했지만, 강의 내용과 비추어봤을때 모니터의 내용과 일치하여 첨부한다.  
앞서 다루었던 말들이 어렵고 복잡해 보였지만, 간단하다. 영상에서 나오는 식당(OS)과 손님(Process)의 예시로 상황을 정리해보자.

**식당에 손님이 와서 식사를 한다고 가정하자**
- 식당에 손님이 식사를 할 수 있도록 4개의 테이블이 준비되어있다.(S=4)
- 손님이 차례로 찾아와서, 테이블에 앉아 식사를 한다(S--)
- 다음 손님이 오면 입구에서 만석이라고 알린다(`wait()`)
- 식사를 마친 손님은 식당을 나가며, 식당에선 다음 손님을 받을 테이블이 준비된다.(S++)
- 식당 주인은 대기중인 손님을 입장시킨다(`signal()`)

위에 이야기를 **식당=OS, 손님=Process**로 대입하여 생각하면 쉽다.

앞서 세마포어로 어렵게 관리해야했던 동기화 문제를 `모니터`라는 제한 장치를 통해서, 관리한다면 개발자의 실수를 최소화하고 시스템의 제어로 비교적 간단하게 해결할 수가 있다.

*wait, signal은 자바의 wait, notify를 의미한다*

### 동기화 문제 간단 정리
#### 동기화 문제를 위한 3가지 전략
- Mutual Exclusion(Mutex)
  - 일종의 Locking 매커니즘, Lock가지고 있을때만 공유 자원에 접근하고 외부에서 접근을 제한함. 자신의 처리가 끝난 경우 Unlock을 해서 다른 프로세스의 접근을 허용
- Semaphore
  - 동시에 리소스에 접근할 수 있는 허용 가능한 갯수를 관리함
    - Binary Semaphore : 2개(0,1)로 허용/비허용만 카운팅 (=> mutex와 유사한 개념)
    - Counting Semaphore : 2개 이상 다수의 프로세스 관리 카운팅
- Monitor
  - Mutex의 Lock기능과 Condition Variables(일종의 Queue)를 가지고 동기화를 관리
    - 임계영역에서 Lock으로 제한하고, 그 뒤에 접근을 시도하는 프로세스는 줄을 세워(Queue) 순차적으로 접근을 관리함

---

*Reference*

- Operating System Concepts (Paperback / 9th Ed.) Books
- http://www.kocw.net/home/search/kemView.do?kemId=1046323
- https://ggodong.tistory.com/97
- https://about-myeong.tistory.com/34