---

title: "ArrayList와 LinkedList"
categories: "dev"
tags:
  - data structure
  - list
---

>이 포스팅은 한빛미디어의 '자바로 배우는 핵심 자료구조와 알고리즘'의 내용을 참고/인용하여 정리한 내용입니다.

## ArrayList vs LinkedList
Java Collection Framework(JCF)는 List라는 interface를 통해 ArrayList와 LinkedList를 제공한다. 

두 클래스의 차이를 자세히 살펴보기 위해선 다음의 링크를 참고하자 [ArrayList와 LinkedList 차이](http://www.nextree.co.kr/p6506/)

정말 간단하게만 둘의 차이를 요약하자면, <code>ArrayList</code>는 이름(Array)에서 볼 수 있듯이, index를 담고 있어서 특정 인덱스를 참고하고자 할 때 검색이나 접근에 용이하다.

반면 리스트 중간의 값이 제거/삽입 되는 경우 중복되거나 비어있는 인덱스를 허용하지 않기 때문에 모든 배열 값들의 이동이 일어난다. 따라서 빈번한 제거/삽입의 경우에는 비효율적인 구조라 볼 수 있다.


<code>LinkedList</code>는 역시 이름(Linked)에서 볼 수 있듯이, 줄줄이 노드가 연결된 구조이다.
따라서 LinkedList는 모든 객체들이 줄줄이 연결된 구조이기 때문에 검색(외부에서 특정 객체에 대한 접근)은 효율적이지 못하다.

하지만 제거/삽입의 경우 전체 노드의 구성은 변함이 없다. 이벤트가 발생한 해동 노드의 주소지만 알고 있다면, 그 위치에 제거/삽입만 진행하면된다. 전체 노드의 구조는 변함이 없다.

### [정리]

#### 검색
데이터 검색 시에는 ArrayList는 LinkedList에 비해 굉장히 빠르다. ArrayList는 인덱스 기반의 자료 구조이며 get(int index) 를 통해 O(1) 의 시간 복잡도를 가진다. 그에 비해 LinkedList는 검색 시 모든 요소를 탐색해야 하기 때문에 최악의 경우에는 O(N)의 시간 복잡도를 가진다.

#### 삽입, 삭제

LinkedList에서의 데이터의 삽입, 삭제 시에는 ArrayList와 비교해 굉장히 빠른데, LinkedList는 이전 노드와 다음 노드를 참조하는 상태만 변경하면 되기 때문이다. 삽입, 삭제가 일어날 때 O(1)의 시작 복잡도를 가진다. 반면 ArrayList의 경우 삽입, 삭제 이후 다른 데이터를 복사해야 하기 때문에 최악의 경우 O(N) 의 성능을 내게 된다.




>단순하게 생각하면 쉬운 것 같다.<br/>
우리는 귀찮거나 반복되거나 하는 비생산적인 일을 싫어하는 프로그래머다.<br/>
극도의 효율을 중시하는 프로그래머가 사용하고, 만든 언어인 만큼 이 두 개념 역시 서로의 부족한 부분에 대한 효율을 채우기 위하여 구분된 클래스이다.<br/>
따라서 한가지 개념만 완벽하게 이해한다면, 그와 비슷한 기능을 하는 다른 클래스/객체에 대한 이해는 덤으로 딸려 올 것이다.<br/>
<code>Stringbuilder</code>와 <code>StringBuffer</code>도 그랬던 것 처럼.


---

## List Interface
ArrayList와 LinkedList는 <code>List</code>를 베이스로 하여 동작하는 클래스이다.

따라서 다음과 같은 구조로 사용할 수 있다.

~~~java
public class ListClientExample {
    private List list;

    public ListClientExample(){
        list = new LinkedList();
    }

    private List getList(){
        return this.list;
    }

    public static void main(String[] args) {
        ListClientExample lce = new ListClientExample();
        List list = lce.getList();
        System.out.println(list);
    }
}
~~~

이 클래스의 구성은 유용하진 않지만, List를 캡슐화하는 클래스의 필수 요소를 가지고 있다.

위 클래스는 생성자를 통해 LinkedList를 초기화하고, getList를 통해 List객체애 대한 참조를 반환한다.

이와 같은 방식을 **인터페이스 프로그래밍**이라고 하는데, 필요한 경우가 아니라면 LinkedList나 ArrayList를 직접 구현하지 않고 생성자를 통해 구현함으로써, 필요에 따라 생성자의 초기화만 변경하여 유연한 대처를 할 수 있다.


[교재 예제 삽입]

[참고]
- https://www.holaxprogramming.com/2014/02/12/java-list-interface/
- 