---
title: "[Operating System] Ch10. File Systems #2"
categories: "OS"
tags:
  - Operating System
  - File System
  - VFS
  - NFS
  - Page cache
  - Buffer cache
---
# File System #2
## Virtual File System & Network File System
### Virtual File System(VFS)
- 서로 다른 다양한 file system에 대해 동일한 시스템 콜 인터페이스(API)를 통해 접근할 수 있게 해주는 OS의 layer
  - UNIX, NFS, 기타 다른 시스템에 속한 파일들을 접근하기 위하여, 개별 파일시스템의 윗 계층에 VFS인터페이스를 통하여 사용자는 파일 시스템에 접근시 VFS 인터페이스를 통해 시스템이 어떤 파일시스템을 사용하는지 의존성을 띄지 않음

### Network File System(NFS)
- 분산 시스템에서는 네트워크를 통해 파일이 공유될 수 있음
  - 로컬 환경이 아닌 원격 시스템에 파일이 존재
- NFS는 분산 환경에서의 대표적인 파일 공유 방법

![](/assets/images/study/dev/2021/os/ch10_vfs-nfs.png)

그림은 좌측에 **client**와 우측의 **server** 2대의 분리된 컴퓨터가 존재한다.  
클라이언트는 자신의 로컬 환경에서 파일을 접근할 때, `VFS Interface`를 통해서 로컬 환경의 파일에 접근할 것이고,  
분리되어 있는 서버의 파일을 접근할 때 `NFS Interface`를 통해서 접근할 수 있다.

## Page Cache & Buffer Cache
![](/assets/images/study/dev/2021/os/ch10_page-cache-buffer-cache.png)
### Page Cache
- Virtual memory의 paging system에서 사용하는 page frame을 caching의 관점에서 설명하는 용어
- Memory-Mapped I/O를 쓰는 경우 file의 I/O에서도 page cache 사용

### Buffer Cache
- 파일시스템을 통한 I/O 연산은 메모리의 특정 영역인 buffer cache 사용
- File 사용의 locality 활용
  - 한번 읽어온 block에 대한 후속 요청시 buffer cache에서 즉시 전달
- 모든 프로세스가 공용으로 사용
- Replacement algorithm필요 (LRU, LFU 등)

### Memory-Mapped I/O
- File의 일부를 virtual memory에 mapping 시킴
- 매핑시킨 영역에 대한 메모리 접근 연산은 파일의 입출력을 수행하게 함

### Unified Buffer Cache
- 최근 OS에서는 기존의 buffer cache가 page cache에 통합됨

![](/assets/images/study/dev/2021/os/ch10_unified-buffer-cache.png)

#### Unified Buffer Cache를 이용하지 않는 File I/O
- 전통적인 파일 I/O 하는 방식
  1. OS는 시스템콜을 통해 파일의 내용이 버퍼캐시에 있는지 확인
     - 버퍼캐시에 파일이 존재하는 경우
        1. 메모리에 전달
     - 버퍼캐시에 파일이 존재하지 않는 경우
        1. 디스크에서 파일을 읽어 옴
        2. 메모리에 전달
  2. 사용자 프로그램에 파일 내용 전달
    1. 사용자 프로그램 자신의 Page 영역에 버퍼캐시의 파일을 copy
    2. 이후에 같은 요청이 오는 경우 해당 Page 영역을 통해서 파일을 접근

- Memory-mapped I/O 를 사용하는 경우
  - OS에 **Memmory-Mapped I/O 시스템 콜**
  - 최초 버퍼캐시에 파일을 copy 하는 과정까지는 동일
  - 이후 버퍼캐시의 내용을 페이지캐시에 copy하여 사용자 프로그램 메모리에 보관
    - 같은 요청이 오는 경우 사용자 프로그램은 Page Cache를 직접 접근(**시스템 콜을 거치지 않음**)

#### Unified Buffer Cache를 이용하는 File I/O
기본적인 File I/O 방식은 모두 동일하지만, **page cache-buffer cache**간의 분리를 하는 것이 아닌,  
page cache에서 직접 I/O를 거치게 되므로 buffer cache 단계를 패싱함  
(Buffer cache <-> Page Cache간의 copy하는 오버헤드를 줄임)


---

*Reference*

- Operating System Concepts (Paperback / 9th Ed.) Books
- https://jhi93.github.io/category/os/2019-12-28-operatingsystem-10-3/