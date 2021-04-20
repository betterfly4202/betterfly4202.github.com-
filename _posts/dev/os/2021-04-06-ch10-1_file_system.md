---
title: "[Operating System] Ch10. File Systems"
categories: "OS"
tags:
  - Operating System
  - File System
---
# File System
## File and File System
- **File**
  - *"A named collection of releated information"*
  - 일반적으로 비휘발성 보조기억장치에 저장
  - 운영체제는 다양한 저장 장치를 file이라는 동일한 논리적 단위로 볼 수 있게 해줌
  - Operation 
    - create, read, write, reposition(lseek), delete, open, close 등

- **File Attribute(혹은 파일의 metadata)**
  - 파일 자체의 내용이 아니라 파일을 관리하기 위한 각종 보조 정보들
    - 파일 이름, 유형, 저장된 위치, 파일 사이즈
    - 접근 권한(읽기/쓰기/실행), 시간(생성/변경/사용), 소유자 등

- **File System**
  - 운영체제에서의 파일을 관리하는 부분
  - 파일 및 파일의 메타데이터, 디렉토리 정보 등을 관리
  - 파일의 저장 방법 결정
  - 파일 보호 등

## Directory and Logical Disk
- **Directory**
  - 파일의 메타데이터 중 일부를 보관하고 있는 일종의 특별한 파일
  - 그 디렉토리에 속한 파일 이름 및 attribute들
  - Operation
    - search for a file, create a file, delete a file
    - list a directory, rename a file, traverse the file system
- **Partition(=Logical Disk)**
  - 하나의 물리적 디스크 안에 여러 파티션을 두는게 일반적  
  (ex. 디스크 하나를 C드라이브, D드라이브로 나눌 수 있음)
  - 여러 개의 물리적인 디스크를 하나의 파티션으로 구성하기도 함
  - 물리적 디스크를 파티션으로 구성한 뒤 각각의 파티션에 *file system*을 깔거나 *swapping*등 다른 용도로 사용할 수 있음

## Open
![](/assets/images/study/dev/2021/os/ch10_file-open.png)

- open("a/b/c")
  - 디스크로부터 파일 c의 메타데이터를 메모리로 가지고 옴
  - 이를 위하여 directory path를 search
    1. 루트 디렉토리 "/"를 open하고 그 안에서 파일 "a" 위치 획득
    2. 파일 "a"를 open 후 read하여 그 안에서 파일 "b"의 위치 획득
    3. 파일 "b"를 open 후 read하여 그 안에서 파일 "c"의 위치 획득
    4. 파일 "c"를 open
  - Directory path의 search에 너무 많은 시간 소요됨
    - open을 read / write와 별도로 두는 이유임
    -  한번 open한 파일은 read / write 시 directory search 불필요

![](/assets/images/study/dev/2021/os/ch10_file-open-process.png)

## File Protection
- 각 파일에 대해 누구에게 어떤 유형의 접근(read/write/execution)을 허락할 것인가?
- Access Control 방법
  - Access control matrix  
  ![](/assets/images/study/dev/2021/os/ch10_access-matrix.png)
    - *Access control list* : 파일별로 누구에게 어떤 접근권한이 있는지 표시
    - *Capabillity* : 사용자별로 자신이 접근 권한을 가진 파일 및 해당 권한 표시
  - Grouping
    - **가장 일반적으로 사용하는 방법**
    - 전체 user를 owner, group, public의 세 그룹으로 구분
    - 각 파일에 대해 세 그룹의 접근 권한(rwx)를 3비트씩으로 표시
    - 예 UNIX
      ![](/assets/images/study/dev/2021/os/ch10_unix-file-grouping.gif)
  - Password
    - 파일마다 password를 두는 방법(디렉토리 파일에 두는 방법도 가능)
    - 모든 접근 권한에 대해 하나의 password : all-or-nothing
    - *접근 권한별 password : 관리 문제 발생*

## File System의 Mounting
root파일 시스템의 특정 directory에 다른 파티션 file system을 `mount`해주면 root 디렉토리를 접근할때마다 다른 파일 시스템의 root 디렉토리를 접근하는 방법이다. (일종의 tree구조)
![](/assets/images/study/dev/2021/os/ch10_file-system-mounting.jpeg)

## Access Methods
- 시스템이 제공하는 파일 정보의 접근 방식
  - 순차 접근(sequential access)
    - 카세트 테이프를 사용하는 방식처럼, 건너띄거나 넘어갈 수 없고 순차적으로 접근 하는 방식
    - 읽거나 쓰면 `offset`은 자동적으로 증가
  - 직접 접근(direct access, random access)
    - LP레코드 판, CD와 같이, 직접 접근하도록 함
    - 파일을 구성하는 레코드를 임의의 순서로 접근할 수 있음

## Allocation of File Data in Disk
임의의 크기를 파일을 동일한 크기의 block 단위로 쪼개서 디스크에 저장하고 있다.
### Contiguous Allocation
![](/assets/images/study/dev/2021/os/ch10_contiguous-allocation.jpeg)
위 그림은 하나의 디렉토리 안에 5개의 파일(count, tr, mail, list, f)이 존재한다.  
디렉토리는 각 파일의 메타데이터(이름, 위치정보 등)를 갖고 있게 된다.

`Contiguous Allocation`은 하나의 파일을 디스크상에 연속된 블록을 점유하는 것을 말한다.  
그림과 같이 하나의 파일의 시작<sub>start</sub>지점부터 블록이 점유하는 길이<sub>length</sub>까지 연속적으로 디스크에 할당된 형태이다.

이 방식의 **단점**으로는,  
- 외부조각<sub>external fragmentation</sub>이 발생할 수 있다는 것이다.  
  예를들어 block의 길이가 3인 파일이 생성된다면 블록 17~18 사이에는 연속된 3개의 블록을 할당받을 수 없으므로 외부조각이 발생하게 된다.
- 파일의 크기가 증가할 경우 제약이 존재하는 것이다.  
  그림의 `tr`파일(블록14~16점유)의 사이즈가 커져서 블록을 3개 이상 필요 한다고 했을때, 연속된 블록을 할당받을 수 없기 때문에 크기 증가가 어렵다.  
  반대로 이런 경우를 대비해서 실제로는 5개의 블록만 사용하면 되는데 여분을 위하여 10개의 조각을 할당받았다고 하자.  
  하지만 실제로 최대 7개까지만 블록을 사용한다면 3개의 낭비, 내부조각<sub>internal fragmentation</sub>가 발생할 것이다.

하지만 이런 방식에 **장점**도 존재한다.
- 빠른 I/O가 가능하다.
  - 하드디스크의 경우 대부분의 접근시간이 head가 이동하는데 소요된다.
  그런데 연속되어 할당된 이 방식의 경우 최초의 시작지점만 seek하면, 그 다음 연속된 블록을 순회하여 한번에 많은 양의 데이터 양을 확보할 수 있다.
  - 위와 같은 장점으로 Realtime file용으로 또는 이미 run중이던 process의 swapping용
- Direct access(=random access)가능
  - 해당 파일의 시작지점<sub>start</sub>을 알기 때문에, 시작지점부터 접근하려는 block의 위치를 더해서 원하는 위치에 직접 접근할 수 있다.

### Linked Allocation
![](/assets/images/study/dev/2021/os/ch10_linked-allocation.png)
`Linked-allocation`은 앞서 연속할당과 달리 각 블록은 비어있는 공간을 점유하고, 각 블록마다 다음 블록의 위치를 기록하여 연결된 형태를 보인다.  
즉 전체 디스크 중 비어있는 공간을 임의로 할당하여 서로 꼬리에 꼬리를 무는 연속된 형태로 파일을 구성한다.  
그리고 더이상 할당된 블록이 없다면, 마지막에 음수(-1)로 표기하여 종료를 알린다.  

디렉토리는 이 파일의 시작위치만 갖고 있고, 구성된 각 블록의 위치는 앞선 블록의 위치에서 직접 확인이 가능하다.

연속할당과 달리, 임의의 공간을 블록 낱개 단위로 접근하기 떄문에 이 방식의 **장점**으로는
- 외부조각<sub>External fragmentation</sub>이 발생하지 않는다.

하지만 그만큼 **단점**이 존재하는데,
- 직접 접근이 불가능하다.(No random access)
  - 예를들어 `jeep`파일의 4번째 블록을 확인하고 싶다면? 직접 4번째 블록의 위치를 확인할 수 없기 때문에 파일의 시작지점부터 4번째 블록을 탐색하여 확인할 수 있다.
- Reliability문제
  - 블록간 연결된 형태이기 때문에, 하나의 sector에 문제가 생길 경우 pointer가 유실되면서 그 뒤에 연결되어 있는 많은 블록 정보를 잃게 된다.
- Pointer를 위한 공간이 block의 일부가 되어 공간 효율성이 떨어뜨림
  - 일반적으로 디스크에서 하나의 sector는 512bytes로 구성
  - sector안에 다음 sector를 가르키는 pointer의 크기는 4bytes
  - 디스크에 저장시 512bytes 단위로 저장을 요청하는데, 실제 포인터의 크기를 제외하면 508bytes만을 저장할 수 있기 때문에,  
  512bytes저장을 해서 실제로 하나의 sector만 사용해도 되지만, pointer가 점유한 크기때문에 2개의 sector를 사용하는 낭비가 발생한다.

#### 변형
- File-allocation table(FAT) 파일 시스템
  - 포인터를 별도의 위치에 보관하여 reliability와 공간 효율성 문제를 해결


### Indexed Allocation
![](/assets/images/study/dev/2021/os/ch10_indexed-allocation.png)
`Indexed Allocation`방식은 각 파일에서 직접 자신의 block정보를 갖고 있는것이 아니라, **index block**이라는 별도의 공간에 자신의 블록 정보를 담고 있는 것이다.  
따라서 디렉토리는 그 파일의 **index block**위치를 알고 있어서, index block에 포함된 내용을 통해서 파일을 확인할 수 있다.  

index block안에 메타 정보를 포함하기 때문에 원하는 위치에 직접 접근도 가능하며,  
연속되어 블록을 점유하는것이 아닌, 임의의 비어있는 공간을 사용하기 때문에 linked-allocation의 이점을 누릴 수 있다.

**장점**
- 외부조각<sub>External fragmentation</sub>이 발생하지 않음
- 직접접근<sub>Direct access</sub> 가능
  
**단점**
- 작은 사이즈의 파일인 경우 공간 낭비(실제로 대다수의 파일들이 작은 파일로 구성되어 있음)
  - 아무리 작은 파일의 경우라도 최소 2개이상의 블록을 사용함(*data block + index block*)
- 매우 큰 사이즈의 파일의 경우 하나의 block으로 index를 저장하기에 부족함
  - 해결 방안
    - linked scheme
    - muliti-level index

### UNIX File System
![](/assets/images/study/dev/2021/os/ch10_unix-filesystem-structure.png)
- Boot block
  - 부팅에 필요한 정보(bootstrap loader)
  - 모든 OS의 메모리 첫번째(0번째 index)에 위치함 -> 부팅시 참조할 공간
    - boot block의 부팅정보를 통해 OS의 커널위치를 찾아서 메모리에 올려준 후 실행됨
- Super block
  - 파일 시스템에 관한 총체적인 정보를 담고 있음
    - 어디가 비어있고, 어디가 실제로 파일이 저장되고 사용중인 block인지 등 전반적인 파일 관리
- Inode list
  - File하나 당 갖고있는 meta data를 갖고 있음 -> 파일 하나의 메타 데이터의 집합 = `inode`
  - 파일 이름을 제외한 파일으 모든 메타 데이터를 저장함
  - inode내 direct/indirect 정보가 있는데, 작은 사이즈의 파일인 경우 direct blocks 정보만으로 파일의 위치를 나타내지만  
  파일의 사이즈가 클 경우 single indirect, double indirect등의 계층구조를 통해서 파일의 위치 정보를 보관한다.  
  즉, `Inode list`의 고정된 크기(최소한의 크기)로 파일의 전체 정보를 보유할 수 있도록 유도한다. 
- Data block
  - 파일의 실제 내용을 보관
  - 파일 이름 + inode의 index값을 보유

### FAT File System
![](/assets/images/study/dev/2021/os/ch10_fat-filesystem.png)
- Boot Block
  - UNIX파일 시스템과 마찬가지로 부팅과 관련된 정보를 담고 있다
- FAT
  - File의 메타데이터 중 일부(파일의 **위치 정보**)를 보관
  - Data block에 1:1로 대응되는 index정보를 보유함
  - FAT의 block에 접근시, 해당 block의 다음 index 번호를 저장함 (linked-allocation)
- Root Driectory
- Data block
  - 파일의 이름을 포함한 모든 메타 데이터(접근 권한, 소유주, 사이즈 등) + 파일의 시작 위치(FAT내 보관되어 있는 위치)


## Free-Space Management
이전까지 파일에서 각각 어떻게 블록들을 관리하고 사용하는지를 다루었다면, 이번에 공부할 내용은 비어있는 블록을 어떻게 관리할 것인지에 대한 기술이다.

### Bit map(bit vector)
![](/assets/images/study/dev/2021/os/ch10_free-space-bitmap.jpeg)
`Bit map`방식은 각 블록별로 인덱스(번호)가 존재하며, 각 번호별로 bit를 부여하여 사용/비사용(1/0)으로 구분하는 방식을 말한다.  

- 각 블록별로 bit map을 선정하는 것이기 때문에 disk에 해당 정보를 보유할 수 있는 부가적인 공간이 필요하다
- bit map은 index에 따라 사용 유무를 판별할 수 있기 때문에 연속적으로 가용가능한 블록<sub>free block</sub>의 상태를 찾는데 효과적이다.

### Linked List
![](/assets/images/study/dev/2021/os/ch10_free-space-linked-list.png)

앞서 비어있는 블록의 포인터를 통해 다음 비어있는 블록의 헤드값을 보유하여 연결된 구조이다.

- 모든 free block들을 링크로 연결(free list)
- 연속적인 가용공간을 찾는 것이 쉽지 않다.
  - 각 블록을 하나씩 탐색해야만 다음 블록의 위치를 파악할 수 있음
- 공간의 낭비가 없다

### Grouping
![](/assets/images/study/dev/2021/os/ch10_free-space-grouping.png)
- linked list 방법의 변형
- 첫번째 free block이 n개의 pointer를 가짐
  - n-1 pointer는 free data block을 가리킴
  - 마지막 pointer가 가리키는 block은 또다시 n pointer를 가짐


### Counting
- Grouping 방식처럼 비어있는 블록의 포인터 위치만 가리키는 것이 아닌,  
그 안에 몇개의 블록이 비어있는지의 수까지 보관함
  - 이를 통해, 5개의 비어있는 블록을 찾고 싶다고하면 그룹핑된 블록 갯수 중 5개의 비어있는 블록을 찾으면 효율적임
- 프로그램들이 종종 여러개의 연속적인 block을 할당하고 반납한다는 성질에 착안
- free free block, # of contiguous free blocks를 유지

## Driectory Implementation
`Directory`는 디렉토리 **하위의 파일에 대한 메타데이터를 관리**하는 특별한 파일구조이다.

![](/assets/images/study/dev/2021/os/ch10_directory-implementation.png)

- 보관 방법
  - **Linear list**
    - `file name`, `metadata`의 list
    - 구현이 간단함
    - 디렉토리 내에 파일이 있는지 찾기 위해서는 linear search 필요 (time-consuming)

  - **Hash Table**
    - linear list + hashing
    - Hash table은 `file name`을 파일의 linear list의 위치로 바꾸어 줌
    - search time을 없앰
    - Collision 발생 가능

- File metadata의 보관 위치
  - 디렉토리 내에 직접 보관
  - 디렉토리에는 포인터를 두고 다른 곳에서 보관
    - inode(UNIX System), FAT 등
  
- Long file name의 지원
  - `file name`, `file metadata`의 list에서 각 entry는 일반적으로 고정 크기
  - file name이 고정 크기의 entry 길이보다 길어지는 경우 entry의 마지막 부분에 이름의 뒷부분이 위치한 곳의 포인터를 두는 방법
  - 이름의 나머지 부분은 동일한 directory file의 일부에 존재


---

*Reference*

- Operating System Concepts (Paperback / 9th Ed.) Books
- https://developerhenrycho.tistory.com/23