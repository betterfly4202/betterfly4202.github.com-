---
title: "[Operating System] Ch4. Process Management"
categories: "OS"
tags:
  - Operating System
  - Process
  - Process Management
---

# Process Management
## Process Creation
- 부모 프로세스(Parent Process)가 자식 프로세스(Children Process)생성 (`COW(Copy-on-write)` write가 발생시 내용이 수정되는 경우 자식 프로세스를 생성)
- 프로세스의 트리(계층 구조) 형성
- 프로세스는 자원(CPU, 메모리 등)을 필요로 함
  - 운영체제로부터 받음
  - 부모와 공유
- 자원의 공유(code, data, stack을 그대로 복사)
  - 부모와 자식이 모든 자원을 공유하는 모델
  - 일부를 공유하는 모델
  - 전혀 공유하지 않는 모델
- 수행(Execution)
  - 부모와 자식은 공존하며 수행되는 모델
  - 자식이 종료(terminate)될 때까지 부모가 기다리는(wait) 모델
- 주소 공간(Address Space)
  - 자식은 부모의 공간을 복사함(binary and OS data)
  - 자식은 그 공간에 새로운 프로그램을 올림
- 유닉스의 예
  - `fork()` 시스템 콜이 새로운 프로세스를 생성
    - 부모를 그대로 복사(OS data except PID + binary)
    - 주소 공간 할당
  - fork 다음에 이어지는 `exec()`시스템 콜을 통해 새로운 프로그램을 메모리에 올림
    - 일단 복제 생성 후 (fork) 필요한 부분은 덮어 씌움(exec)

### fork()시스템 콜
프로세스는 `fork()`라는 시스템콜을 통하여 생성된다.(새로운 주소 공간<sub>address space</sub>은 시스템콜러로 부터 복제된다.)

```c
  int main(){
    int pid;
    pid = fork();
    if(pid == 0) // child
      printf("\n Heelo, Iam child!\n"); // child process의 코드실행 영역
    else if(pid > 0) // parent
      printf("\n Hello, I am parent");  // parent process의 코드실행 영역
  }
```

예를들어 fork()를 하는 부모프로세스에서 fork()가 발생하는 경우 자식 프로세스는 부모의 모든 코드 영역을 복제하는 것이 아니라, 
부모의 PC(Program Counter)부분 중 기억하여, 자식 프로세스는 fork() 다음 부분을 복제하게 된다.(코드상 if문 이후의 소스코드)  
fork를 수행후 부모 프로세는 **pid값이 양수**로 리턴하지만, 자식프로세스는 **pid 값이 0**으로 고정되어 구분이 가능하다.

### exec() 시스템 콜
앞서 살펴본 fork()를 통한 자식프로세스가 생성되면, 모든 자식프로세스는 동일한 로직만 수행하게 된다.  
`exec()`는 프로그램을 새로운 프로세스로 태어나게 해주는 역할을 해준다.

```c
  int main(){
    int pid;
    pid = fork();
    if(pid == 0){ // child
      printf("\n Heelo, Iam child! Now I'll run date \n"); // child process의 코드실행 영역
      execlp("/bin/date", "/bin/date", (char*)0);
    } 
      
    else if(pid > 0) // parent
      printf("\n Hello, I am parent");  // parent process의 코드실행 영역
  }

  // ex) "/bin/date" 영역의 코드 샘플
  // 자식 프로세스를 통해 실행된 /bin/date의 코드는 argument (char*0)를 전달받아 실행할 수 있다.
  int main(){
    ...
  }
```

예를들어 다음과 같은 코드가 있다면,

```c
  int main(){
    printf("\n Heelo, I am child! \n"); 
    execlp("/bin/date", "/bin/date", (char*)0);
    printf("\n next to exec()");
```

위 코드 중 앞서 exec()을 통해서 프로세스가 이관되었기 때문에 마지막에 위치한 *printf("\n next to exec()");* 코드는 실행되지 않는다.

## Process Termination
- 프로세스가 마지막 명령을 수행한 후 운영체제에 리르 알려줌(exit)
  - 자식이 부모에게 output data를 보냄 (via wait)
  - 프로세스의 각종 자원들이 운영체제에게 반납됨
- 부모 프로세스가 자식의 수행을 종료시킴(abort)
  - 자식이 할당 자원의 한계치를 넘어섬
  - 자식에게 할당된 태스크가 더 이상 필요하지 않음
  - 부모가 종료(exit)하는 경우
    - 운영체제는 부모 프로세스가 종료하는 경우 자식이 더 이상 수행되도록 두지 않는다
    - 단계적인 종료

### wait() 시스템 콜
- 프로세스 A가 wait()시스템 콜을 호출하면
  - 커널은 child가 종료될 때까지 프로세스A를 sleep시킨다 (block 상태)
  - child process가 종료되면 커널은 포르세스 A를 깨운다 (ready 상태)

![](/assets/images/study/dev/2021/os/ch4_wait_systemcall.png)

### exit() 시스템 콜
- 프로그램을 종료시킬때 사용하는 시스템 콜

```c
  int main(){
    printf("\n Heelo, I am child! \n"); 
    exit() // 프로그램 즉시 종료
    execlp("/bin/date", "/bin/date", (char*)0);
    printf("\n next to exec()");
```

- 프로세스의 종료
  - 자발적 종료
    - 마지막 statement 수행 후 exit() 시스템 콜
    - 프로그램에 명시적으로 적어주지 않아도 main함수가 리턴되는 위치에 컴파일러가 넣어줌(main함수 전체 코드 수행 후 자동으로 exit() 수행됨)
  - 비자발적 종료
    - 부모 프로세스가 자식 프로세스를 강제 종료시킴
      - 자식 프로세스가 한계치를 넘어서는 자원 요청
      - 자식에게 할당된 태스크가 더 이상 필요하지 않음
    - 키보드로 kill(`crtl+c`), break 등을 입력한 경우
    - 부모가 종료하는 경우
      - 부모 프로세스가 종료하기 전에 자식들이 먼저 종료됨

## 프로세스와 관련된 시스템 콜
- fork()
  - create a child(`copy`)
- exec()
  - overlay new image
- wait()
  - sleep until child is done
- exit()
  - frees all the resoucres, notify parent

## 프로세스 간 협력
- 독립적 프로세스(Independent process)
  - 프로세스는 각자의 주소 공간을 가지고 수행되므로 원칙적으로 하나의 프로세스는 다른 프로세스의 수행에 영향을 미치지 못함
- 협력 프로세스(Cooperating process)
  - 프로세스 협력 메커니즘을 통해 하나의 프로세스가 다른 프로세스의 수행에 영향을 미칠 수 있음
- 프로세스 간 협력 메커니즘(**IPC: Interprocess Communication**)
  - 메시지를 전달하는 방법
    - message passing : 커널을 통한 메시지 전달
  - 주소 공간을 공유하는 방법
    - shared memory : 서로 다른 프로세스 간에도 일부 주소 공간을 공유하게 하는 shared memory 메커니즘
    - `thread`
      - **thread**는 사실상 하나의 프로세스이므로, 프로세스 간 협력으로 보기는 어렵지만 동일한 process를 구성하는 thread들 간에는 주소 공간을 공유하므로 협력이 가능함

### Message Passing
- Message system
  - 프로세스 사이에 공유변수(shared variable)를 일체 사용하지 않고 통신하는 시스템
- Direct Communication
  - 통신하려는 프로세스의 이름을 명시적으로 표시
![](/assets/images/study/dev/2021/os/ch4_direct_comm.png)
- Indirect Communication
  - mailbox(또는 port)를 통한 메시지 간접 전달
![](/assets/images/study/dev/2021/os/ch4_indirect_comm.png)

### Interprocess Communication
![](/assets/images/study/dev/2021/os/ch4_inter-process-communications-models.png)

> 임의로 `shread memory`영역을 구분하여 공유되는것이 아니라, 커널에게 shared memory를 공유하겠다는 시스템 콜을 통해서 공유된 공간을 맵핑 후 자원을 공유할 수 있다.



---

*Reference*

- Operating System Concepts (Paperback / 9th Ed.) Books
- http://www.kocw.net/home/search/kemView.do?kemId=1046323