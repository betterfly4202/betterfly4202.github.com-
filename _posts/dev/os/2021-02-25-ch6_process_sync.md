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
#### Mutual Exclusion
프로세스 P<sub>i</sub>가 critical section 부분을 수행 중이면 다른 모든 프로세스들은 그들의 critical section에 들어가면 안된다.

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

#### 내용 추가
![](https://www.youtube.com/watch?v=PQ5aK5wLCQE)
`세마포어`라는 낯선 개념 때문에 강의를 들으면서 이해가 안되어서 다른 자료를 찾아봤다.  
말이 어렵고 복잡해 보였지만, 간단하다. 식당(OS)과 손님(Process)라는 예시로 이해를 했는데  

**식당에 손님이 와서 식사를 한다고 가정하자**
- 식당에 손님이 식사를 할 수 있도록 4개의 테이블이 준비되어있다.(S=4)
- 손님이 차례로 찾아와서, 테이블에 앉아 식사를 한다(S--)
- 다음 손님이 오면 입구에서 만석이라고 알린다(`wait()`)
- 식사를 마친 손님은 식당을 나가며, 식당에선 다음 손님을 받을 테이블이 준비된다.(S++)
- 식당 주인은 대기중인 손님을 입장시킨다(`signal()`)

위에 이야기를 **식당==OS, 손님==Process**로 대입하여 생각하면 쉽다.


---
##### DeadLock & Starvation



---

*Reference*

- Operating System Concepts (Paperback / 9th Ed.) Books
- http://www.kocw.net/home/search/kemView.do?kemId=1046323
- https://ggodong.tistory.com/97