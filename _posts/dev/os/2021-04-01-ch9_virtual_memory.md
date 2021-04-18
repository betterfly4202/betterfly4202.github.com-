---
title: "[Operating System] Ch9. Virtual Memory"
categories: "OS"
tags:
  - Operating System
  - Virtual Memory
  - vm
---

# Demand Paging
이전에 공부했던 메모리 관리에서 **물리적인 메모리의 주소 변환은 운영체제가 관여하지 않는다**고 했는데 `Virtual Memory`에서는 전적으로 운영체제가 관리하고 있다.

견고한 소프트웨어는 방어적으로 설계되어 있어서 기본 동작에 사용하지 않는 많은 부분의 코드가 대부분을 차지하고 있다. 이런 코드는 일반적인 경우 실행되지 않기 때문에 모든 코드들을 메모리에 올려 놓는 것은 메모리 낭비가 발생할 수 있다.  
`Demand Paging`이란, 이처럼 메모리 낭비를 방지하기 위하여 **요청이 있을때 메모리에 올리는 기법**을 말한다.

- 실제로 필요할 때 page를 메모리에 올리는 것
  - I/O 감소
  - Memory 사용량 감소
  - 빠른 응답 시간
  - 더 많은 사용자를 수용
- Valid/Invalid bit의 사용
  - Invalid의 의미
    - 사용되지 않는 주소 영역인 경우
    - 페이지가 물리적 메모리에 없는 경우
  - 처음에는 모든 page entry가 invalid로 초기화
  - address translation시 invalid bit 이 set되어 있으면 => `page fault` 발생  
  (요청한 페이지가 메모리에 없는 상태, page fault)

![](/assets/images/study/dev/2021/os/ch9_vm-page-table.png)
`logical memory` - `page table` - `physical memory` 흐름은 앞서 살펴본 페이징 구조와 같지만 맨 뒤에 **backing store(=disk)**가 존재한다. (혹은 `swap area`라고 함)  
즉, 당장에 사용되는 필요한 부분은 physical memory에 상주해 있을 것이고, 그렇지 않은(사용되지 않는) 데이터는 backing store에 보관된다.

## Page fault
![](/assets/images/study/dev/2021/os/ch9_page-fault.png)

그림의 순서대로 흐름을 살펴보면,

1. logical memory에서 주소 변환을 하려고 페이지 테이블을 확인해보니 i<sub>invalid</sub>상태(메모리에 올라와 있지 않은 상태)
2. 때문에 **trap**이 발생(`page fault trap`)하여 CPU가 운영체제에게 넘어가게 된다.
3. CPU를 할당받은 운영체제(kernel mode)는 backing store에 있는 해당 page를 탐색한다.
4. backing store에서 확인된 페이지 정보를 physical memory로 이동시킨다.  
  (만약 비어있는 page frame이 없으면, 대체됨)
5. page frame에 정보를 입력 후 페이지 테이블의 엔트리에 저장된 page frame 정보를 입력한 후 bit 상태를 변경한다.  
  invalid -> **valid**
6. 이전에 중지되었던 instruction을 재개한다.

## Performance of Demand Paging
page fault 발생시 disk를 경유하는 작업은 프로그램 입장에서 굉장히 부담스러운(오래 걸리는)작업이다.

- page fault 발생시 overhead
  - OS & HW page fault의 over head
  - page frame에 비어있는 공간이 없을 경우 swap page out 후 해당 공간에 다시 swap page in 하는 과정
  - OS는 해당 페이지 테이블을 valid bit으로 표기 후 OS & HW를 재시작하는 over head

## Free frame이 없는 경우  
**OS에서 처리해주는 과정**  

- Page replacement
  - 어떤 frame을 빼앗아 올지 결정해야함
  - 곧바로 사용되지 않을 page를 쫓아내는 것이 좋음
  - 동일한 페이지가 여러번 메모리에서 쫓겨났다가 다시 들어올 수 있음
- Replacement Algorithm
  - page-fault rate를 최소화하는 것이 목표
  - 알고리즘 평가
    - 주어진 page reference string에 대해 page fault를 얼마나 내는지 조사
    - reference string의 예 :  
    (페이지 참조 순서) 1,2,3,4,1,2,5,1,2,3,4,5

![](/assets/images/study/dev/2021/os/ch9_page-replacement.jpeg)
1. 어떤 걸 제거해야할지<sub>vict</sub> 결정 후 backing store(disk)로 *swap out*
2. swap out된 페이지 프레임을 엔트리에서 invalid bit로 변경
3. backing store에서 메모리로 *swap in*
4. 새로 페이지 프레임에 할당된 정보를 valid bit로 변경



---

*Reference*

- Operating System Concepts (Paperback / 9th Ed.) Books
- https://developerhenrycho.tistory.com/23