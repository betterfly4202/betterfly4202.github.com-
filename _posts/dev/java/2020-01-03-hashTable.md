---
title: "Hash Table"
categories: "Java"
tags:
  - Data Structure
  - Hash Table
---

# Hash란?<br/>
`Hash`, `Hashing`, `Hash Table` 이라고 불리기도 한다.  
결국 **Hash**라는 개념은 데이터의 관리 및 유지 보수를 위한 자료구조이다.  
Hash의 특징은 `Resource`를 활용하여, `속도`를 취하는 것에 있다.  
즉, 여러 자원을 활용하여 물체를 1열로 세우는 것이 아닌, **'바구니(빈에 넣는다 : binning)에 나누어 넣는다'**는 개념에서 출발한다.

`Hash Table`은 값을 담을때 *Hash 함수*를 이용하여 바구니<sub>bin</sub>에 나누어 담는 것을 말하며,  
우리에게 더욱 친숙한 `Hash Map`은 바구니<sub>bin</sub>를 결정할때 쓰는 물체(Key)와 바구니<sub>bin</sub>에 넣는 물체(Value)를 구분하는 것이다.  
특정 값에 대한 해시함수는 언제나 동일한 해시값을 보장하기 때문에, 데이터에 접근시 테이블 크기와 상관없이 상수시간으로 접근이 가능하다.  
즉 해시테이블은 데이터의 조회, 삽입, 삭제 등에서 <span class="mi" id="MathJax-Span-3" style="font-family: STIXGeneral-Italic;">O</span>(1)의 시간 복잡도를 보인다.

그럼 **Hash Table**을 어떻게 구현할 수 있는지 자세히 살펴보자.

# Hash 테이블 만들기  

**A,B,C** 라는 각각의 물체를 어떤 바구니에 담을지 결정하는 과정을 생각해보자.  
알파벳은 다음의 ASCII 값을 갖고 있다.  
- A - 65
- B - 66
- C - 67

위와 같은 규칙으로 나열된 알파벳을 서로 다른 바구니로 관리한다면 더욱 효율적으로 접근할 수 있을 것이다.  
예를들어, **Key%10** 연산으로 해시값을 정의한다면, 10개의 비어있는 바구니를 만들어놓고 알파벳이 들어올때마다 각각의 ASCII코드를 통해 알맞은 바구니에 넣어주면 된다.  
![hashTable](/assets/images/study/dev/2020/01/HashTable_default.jpeg)

이로써 단순하지만 효율적으로 각각의 데이터를 관리할 수 있게 되었다.  
하지만 문제가 있다.  
- 비어있는 배열의 공간보다 더 많은 값이 들어온다면?
- 해시함수로 반환한 값에 중복이 발생하여 같은 방을 향해야 한다면 어떻게 될까?

이러한 문제를 `해시충돌(Hash Collision)`이라고 하며, 이 해시충돌을 다루는 것이 해시테이블의 핵심이다.

## Hash Collision
### Chaining <sub>with Linked List</sub>

각 배열이 하나의 공간이 아닌, *연결리스트(LinkedList)*로 구현되어 있다면 중복된 해시 결과를 만들어내도, 리스트에 각각의 노드를 연결하여 데이터를 관리할 수 있다.

![hashTableChaining](/assets/images/study/dev/2020/01/HashTable_chaining.jpeg)

그림과 같이, `K, U`와 같이 해시값이 **5**로 떨어진다면, 이미 사용중인 5번 공간의 `A`뒤에 줄을 서는 것이다.(LinkedList)  
하지만 이 경우 데이터에 접근하기 위해선 각 배열의 위치에서 리스트를 전부 스캔해야한다.  
그렇다면 해시테이블에서 지향한다던 데이터 접근을 위한 시간 복잡도는 <span class="mi" id="MathJax-Span-3" style="font-family: STIXGeneral-Italic;">O</span>(1) 을 희생할 수 밖에 없다.  
이렇게 된다면 각 배열의 위치에서 데이터의 n개 만큼 조회를 해야하기 때문에 <span class="mi" id="MathJax-Span-3" style="font-family: STIXGeneral-Italic;">O</span>(n)의 시간복잡도를 보이게 된다.

### Open Addressing

앞서 다루었던 `Chaining` 과정중에 만약 운이 나쁘게도 해시함수의 결과가 계속 같은 bin 으로 향하게 한다면 **하나의 bin에 쏠리는 현상**이 발생할 수 있다.  
그 대안으로 나온것 중 가장 간단한 방법인 `선형탐색(Linear Probing)` 기법을 알아보자.  
이 기법은 아주 심플하다. 해시값에 해당하는 공간에 이미 다른 데이터가 있다면, 그 다음 공간으로 이동하고, 또 있으면 또 그 다음 방으로 이동하면서 비어있는 방이 있을때까지 찾아가는 것이다.

![hashTableLinearProbing](/assets/images/study/dev/2020/01/HashTable_linearProbing.jpeg)

그림과 같이 비어있는 공간을 찾아서 자신의 자리를 찾아가는 것이다.  
하지만 *선형탐색*을 구현하는데 한가지 주의할점이 있다.  
이렇게 줄줄이 배열의 공간을 차지하고 있기 때문에, 중간에 `삭제`가 발생할 경우 삭제된 공간에 대한 처리(dummy node)를 신경써야 한다는 것이다.

![HashTable_linear_delete](/assets/images/study/dev/2020/01/HashTable_linear_delete.jpeg)

연결된 배열의 공간으로 데이터를 조회하는데, 그림의 좌측과 같이 7번 공간이 비어있지만 8번 공간부터는 데이터가 존재하는 경우 8번 이후의 데이터까지 도달하지 못한채 조회가 종료되며 데이터가 없다고 판단할 수 있다.  
이처럼 삭제된 공간은 더미 데이터를 넣어주어 전체 데이터구조의 형태를 유지해주는 것이 관건이다.

---

앞서 다루었던 예제는 A,B,C와 같은 단순한 문자 하나로 처리했지만, 이보다 복잡한 *문자열(String)* 또는 *Object* 단위의 객체를 다룰떄도 다르지 않다.  
결국 모든 것은 '숫자'로 통용되기 때문에 **객체를 어떻게 숫자화 시킬 것이냐**를 고민하면 된다.

---

다음은 stack overflow에 소개된 **Hash Function을 구현하는 과정**이다.  
다음 시간은 아래 내용을 참고해서 해시함수를 구현해봐야겠다.

>The best implementation? That is a hard question because it depends on the usage pattern.  
A for nearly all cases reasonable good implementation was proposed in Josh Bloch's Effective Java in Item 8 (second edition). The best thing is to look it up there because the author explains there why the approach is good.  

**A short version**
1. Create a int result and assign a non-zero value.
2. For every field f tested in the equals() method, calculate a hash code c by:
- If the field f is a boolean: calculate (f ? 0 : 1);
- If the field f is a byte, char, short or int: calculate (int)f;
- If the field f is a long: calculate (int)(f ^ (f >>> 32));
- If the field f is a float: calculate Float.floatToIntBits(f);
- If the field f is a double: calculate Double.doubleToLongBits(f) and handle the return value like every long value;
- If the field f is an object: Use the result of the hashCode() method or 0 if f == null;
- If the field f is an array: see every field as separate element and calculate the hash value in a recursive fashion and combine the values as described next.
3. Combine the hash value c with result: <br/> **result = 37 * result + c**
4. Return result

---

**Reference**

- https://youtu.be/S7vni1hdsZE
- https://ratsgo.github.io/data%20structure&algorithm/2017/10/25/hash/
- https://stackoverflow.com/questions/113511/best-implementation-for-hashcode-method-for-a-collection