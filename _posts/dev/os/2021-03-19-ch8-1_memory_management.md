---
title: "[Operating System] Ch8-1. Memory Management"
categories: "OS"
tags:
  - Operating System
  - Memory Management
---

# Memory Management
## Logical vs. Physical Address
- Logical address (=virtual address)
  - 프로세스마다 독립적으로 가지는 주소 공간
  - 각 프로세스마다 0번지부터 시작
  - **CPU가 참조하는 주소**
- Physical address
  - 메모리에 실제 올라가는 위치

> 주소 바인딩 : 주소를 결정하는 것  
Symbolic Address -> Logical Address -> Physical address


## Address Binding
- Compile time binding
  - 물리적 메모리 주소<sub>physical address</sub>가 컴파일시 알려짐
  - 시작 위치 변경시 재 컴파일
  - 컴파일러는 절대 코드<sub>absolute code</sub> 생성
- Load time binding
  - Loader의 책임하에 물리적 메모리 주소 부여
  - 컴팡일러가 재배치 가능 코드<sub>relocatable code</sub>를 생성한 경우 가능
- Exceution time binding(=Run time binding)
  - 수행이 시작된 이후에도 프로세스의 메모리 상 위치를 옮길 수 있음
  - CPU가 주소를 참조할 때마다 binding을 점검 (address mapping table)
  - 하드웨어적인 지원이 필요
    - (e.g., *base* and *limit registers*, *MMU*)

![](/assets/images/study/dev/2021/os/ch8_address_binding.png)

- Symbolic address : 소스코드상 변수 A,B,C와 메서드 Add, Jump를 사용
- Logical address : Symbolic address가 컴파일 된 후 실행을 위하여 논리적인 주소 공간을 할당받음
- Physical address : Logical address가 할당된 후 실제 메모리 공간에 해당 값들이 저장됨
  - Compile time binding : 컴파일 시점에 할당된 논리적 주소값 그대로 메모리에 할당된 후 변하지 않고, 해당 메모리 주소에 고정적으로 사용됨(다른 여유있는 메모리 공간이 있음에도, 논리적 주소로 결정된 위치를 사용하기 때문에 비효율적임)
  - Load time binding : 프로그램 시작 후 메모리에 올라가면서, 메모리에 비어있는 메모리 공간을 확보함
  - Run time binding : 프로그램 시작 후 메모리 공간을 확보한 이후에, 실제 코드 영역이 사용되는 시점에(동적으로) 비어있는 메모리 공간을 확보함  
  *현재의 프로그래밍모델이 채택함*

**CPU가 참조하는 주소는 왜 Logical address일까?**  
예를들어 `add(A,B)`의 연산을 한다고 하자. add 메서드의 logical address는 컴파일되며 20,30의 논리적 주소값을 내포하고 있다.  
이 instruction 코드와 매핑되어있는 논리적인 주소가 실제 물리적인 주소 공간에 위치할 뿐이지, 해당 instruction을 가르키는 컴파일된 주소 공간을 변경하지 않는다.

위에 설명했듯이 물리적인 공간은 언제든지 변경가능하므로, CPU는 변하는 물리적 공간을 참조하는 것이 아니고 컴파일된 instruction의 `절대적인 주소 값`을 참조하는 것이다.

### Memory-Management Unit (MMU)
- `logical address`를 `physical address`로 매핑해주는 하드웨어 기기
- MMU scheme
  - 사용자 프로세스가 CPU에서 수행되며 생성해내는 모든 주소값에 대해 base register(=relocation register)의 값을 더한다
- User program
  - logical address 만을 다룬다
  - 실제 physical address를 볼 수 없으며, 알 필요가 없다

![](/assets/images/study/dev/2021/os/ch8_dynamic-relocation.png)

- 프로세스 P<sub>1</sub>은 가상 메모리<sub>virtual memory</sub>의 0부터 3000 까지의 메모리 공간을 점유하고 있다. (화면 좌측 하단)
- CPU는 논리적 위치 값으로 **346**을 할당한다
- CPU와 메모리사이에 MMU가 위치해 있다
  - MMU내 `relocation register`와 `limit register`를 이용하여 주소 변환을 한다.
- 물리적인 메모리에 올라갈 시작위치(그림상, 14000)와 논리적 위치(346)을 더해준다. (14346)

#### Limit Register
프로그램에서 악의적으로 혹은 잘못된 조작으로 인하여 자신에게 할당된 메모리 영역 외의 공간을 요청할 수 있다.  
예를들어 위 그림상 P<sub>1</sub>은 0~3000까지의 주소 공간을 할당받았는데, 4000영역의 주소 공간을 요청한다면 다른 프로세스에서 점유 중인 영역이거나 해당 프로세스에 할당된 영역이 아니기 때문에 문제가 발생한다.

이런 경우 할당된 영역을 넘지 못하도록 제어하는 장치가 바로 `Limit Register`이다.

![](/assets/images/study/dev/2021/os/ch8_address-transaction.png)

1. CPU가 논리적 주소를 요청
2. `limit register`는 요청받은 주소값을 확인
   1. limit register 값을 벗어나는 요청인 경우
      1.  `trap` 발생(software interrupt) 
      2.  OS에게 제어권이 넘어감
      3.  OS에서 프로그램을 어떻게 처리할 것인지 결정하여 처리
  1.  limit register 범위 내의 요청
      1.  relocation register에게 전달됨
3.  relocation register의 값과 논리적 주소값을 연산함
4.  최종 연산값을 메모리에 등록

## Some terminologies
### Dynamic Loading
- 프로세스 전체를 메모리에 미리 다 올리는것이 아닌, 해당 루틴이 불려질 때 메모리에 로드하는 것
- memory utilization 향상
- 가끔식 사용되는 많은 양의 코드의 경우 유용함
  - 예: 오류 처리 루틴
- 운영체제의 특별한 지원 없이 프로그램 자체에서 구현 가능(OS는 라이브러리를 통해 지원 가능)

### Overlays
- 메모리에 프로세스의 부분 중 실제 필요한 정보만을 올림(Dynamic Loading과 동일한 개념)
- 프로세스의 크기가 메모리보다 클 때 유용
- 운영체제의 지원없이 사용자에 의해 구현
- 작은 공간의 메모리를 사용하던 초창기 시스템에서 수작업으로 프로그래머가 구현
  - "Manual Overlay"
    - 프로그래머가 수작업으로 Manual하게 작업한다는 의미
  - 프로그래밍이 매우 복잡

> **Dynamic Loading vs Overlays**  
기본적인 정의는 같지만, Dynamic Loading은 OS 라이브러리를 통해서 처리하지만  
Overlays는 프로그래머가 직접 구현하여 처리한다

### Swapping
![](/assets/images/study/dev/2021/os/ch8_swapping.jpeg)

- Swapping
  - 프로세스를 일시적으로 메모리에서 backing store로 쫓아내는 것 
- Backing store (=swap area)
  - 디스크
    - 많은 사용자의 프로세스이미지를 담을 만큼 충분히 빠르고 큰 저장 공간
- Swap in / Swap out
  - 일반적으로 중기 스케줄러(swapper)에 의해 swap out시킬 프로세스 선정
  - priority-based CPU scheduling algorithm
    - priority가 늦은 프로세스를 `swapped out` 시킴
    - priority가 높은 프로세스를 메모리에 올려 놓음
  - Compile time 혹은 load time biding에서는 원래 메모리 위치로 `swap in` 해야 함
  - Exceution time binding에서는 추후 빈 메모리 영역 아무곳에서나 올릴 수 있음
  - swap time은 대부분 transfer time(swap되는 양에 비례하는 시간)임

### Dynamic Linking
- Linking을 실행 시간(execution time)까지 미루는 기법
  - **Link**란, 메모리의 여러 군데 산재되어 있는 컴파일된 파일들을 묶어서 하나의 실행 파일로 만드는 것
- Static Linking
  - 라이브러리가 프로그램의 실행 파일 코드에 포함됨
  - 실팽 파일의 크기가 커짐
  - 동일한 라이브러리를 각각의 프로세스가 메모리에 올리므로, 메모리 낭비(eg. printf함수의 라이브러리 코드)
- Dynamic Linking
  - 라이브러리가 실행시 연결(link)됨
  - 라이브러리 호출 부분에 라이브러리 루틴의 위치를 찾기위한 stub이라는 작은 코드를 둠
  - 라이브러리가 이미 메모리에 있으면 그 루틴의 주소로 가고,  
  없으면 디스크에서 읽어옴
  - 운영체제의 도움이 필요


---

*Reference*

- Operating System Concepts (Paperback / 9th Ed.) Books
- https://jhi93.github.io/category/os/2019-12-20-operatingsystem-08-1/