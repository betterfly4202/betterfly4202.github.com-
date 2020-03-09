---
title: "[Think Data Structures]Linked List 구현하기 (indexOf)"
categories: "Java"
tags:
  - Data Structure
toc: true
---

![tds_book](/assets/images/study/dev/2019/tds_book.jpg){: .center}
>이 포스팅은 한빛미디어의 '자바로 배우는 핵심 자료구조와 알고리즘'의 내용을 참고 정리한 내용입니다.
> 소스 참고: https://github.com/yudong80/ThinkDataStructures/blob/master/solutions/src/com/allendowney/thinkdast/MyLinkedList.java

* 목차
{:toc}

# INTRO

이전글 [[Think Data Structures]List 인터페이스의 구현과 알고리즘 분석#2](https://better-dev.netlify.com/dev/tds_ch_3_1/)

지금까지 ArrayList에 대해서 다루었다면, 이번엔 LinkedList를 살펴보자.

우선 LinkedList의 개념부터 살펴봐야하는데, 사실 필자의 경우 실무에서 대부분(이라곤 하지만 100%) ArrayList만을 사용해왔기 때문에 `LinkedList`는 조금 낯설게 느껴진다.

>**LinkedList** <br/>
순차적으로 연결된 ArrayList와는 달리, LinkedList의 핵심은 Node라는 객체들이 다른 노드에 대해서 '참조'하고 있다는 것이다.

(자세한 내용은 [1장](http://www.hanbit.co.kr/channel/category/category_view.html?cms_code=CMS4973879534&cate_cd=001)을 참고)

---

자 이제 본격적으로 `LinkedList`를 파헤쳐보자.

## MyLinkedList 와 MyArrayList

~~~java
public class MyLinkedList<E> implements List<E>{
    private int size;  // 요소의 개수를 추적
    private Node haed; // 첫 번째 노드에 대한 참조

    public MyLinkedList(){
        haed = null;
        size = 0;
    }
}
~~~

클래스 분석에 앞서, ArrayList와 비교해보자.

~~~java
public class MyArrayList<T> implements List<T> {
    private int size;
    private T [] array;
    private final int DEFAULT_CAPACITY = 10;

    @SuppressWarnings("unchecked")
    public MyArrayList(){
        array = (T[]) new Object[DEFAULT_CAPACITY];
        size = 0;
    }
}
~~~

각 클래스의 멤버 변수를 비교해보면 size라는 요소의 개수를 확인할 수 있는 변수는 공통으로 사용된 모습이다.

하지만 **ArrayList**는 `배열`을 컨셉으로 하여, 생성자를 통해 해당 배열을 초기화 해주는 모습(최초 사이즈가 '10'인 배열)이다.

그리고 앞선 ArrayList구현시 set(), indexOf() 등의 메서드를 구현시 `array`라는 변수의 배열을 핸들링했다.

하지만 **LinkedList**는 다르다. Node라는 객체를 선언하여, 해당 객체는 LinkedList의 생성자를 통해 null로 초기화된다.

정리하면 ArrayList는 `배열` 을, LinkedList는 `Node` 라는 객체를 다룬다는 것이다.

어떻게 활용되는지 하나씩 짚어보도록 하자.

---

## MyLinkedList

다시 MyLinkedList로 돌아오자.

우선 `Node` 라는 클래스의 구성이 궁금하다.

책에는 다음과 같이 MyLinkedList에 대한 기본 정보를 제공해준다.

~~~java
public class MyLinkedList<E> implements List<E>{
    private int size;  // 요소의 개수를 추적
    private Node haed; // 첫 번째 노드에 대한 참조

    public MyLinkedList(){
        haed = null;
        size = 0;
    }

    private class Node{
        public E data;
        public Ndoe next;

        public Node(E data) {
            this.data = data;
            this.next = null;
        }

        public Node(E data, Node next){
            this.data = data;
            this.next = next;
        }
    }

    public boolean add(E element){
        if(head == null){
            head = new Node(e);
        }else{
            Node node = head;

            for(; node.next != null; node = node.next){

            }
            node.next = new Node(e);
        }
        size ++;
        return true;
    }
}
~~~

MyLinkedList 클래스 내부에 inner class로 `Node` 클래스가 존재한다.

그리고 add(E) 메서드를 제공받았다.

자 이제 기본 정보는 제공되었다.

테스트 코드의 적신호를 제거하며 본격적으로 구현해보자.

---

# MyLinkedList 구현하기

## indexOf()
처음 진행해볼 것은 `indexOf` 메서드이다.

앞서 ArrayList에서 [google docs의 명세](https://docs.oracle.com/javase/8/docs/api/java/util/List.html#indexOf-java.lang.Object-)는 살펴보았기 때문에 해설은 생략한다.

>**HINT** <br/>
*null 을 어떻게 처리할지 고민해보세요.*

### Test Code
테스트 코드는 ArrayList에서 진행했던 것과 동일한 내용이다.

당연히 `List`라는 동일한 인터페이스를 구현하는 것이기 때문에 구현 결과는 동일해야 한다.

~~~java
public class MyLinkedListTest {
    protected List<Integer> mylist;

	/**
	 * @throws java.lang.Exception
	 */
	@Before
	public void setUp() throws Exception {
		mylist = new MyLinkedList<Integer>();
		mylist.add(1);
		mylist.add(2);
		mylist.add(3);
	}

    @Test
    public void testIndexOf() {
        assertThat(mylist.indexOf(1), is(0));
        assertThat(mylist.indexOf(2), is(1));
        assertThat(mylist.indexOf(3), is(2));
        assertThat(mylist.indexOf(4), is(-1));

        assertThat(mylist.indexOf(7), is(-1));
    }
}
~~~

### 구현하기

ArrayList는 하나의 배열 안에 인덱스를 하나씩 갖고와서 객체를 비교 했다면,

이번엔 Node라는 객체를 기준으로 그 객체의 data를 하나씩 검증해야 한다. 그리고 이 객체는 특정 index를 한번에 표현할 수 없고,

현재 객체의 다음 (next 메서드 호출) 객체만 확인이 가능하기 때문에 순차적으로 검증해야한다.

구현은 다음과 같이 진행했다.

~~~java
    @Override
    public int indexOf(Object o) {
        Node node = head;

        for (int i=0; node != null; i++){
            if(node.data.equals(o)){
                return i;
            }
            node = node.next;
        }

        return -1;
    }
~~~

궁극적으로 해당 노드의 data값이 파라미터로 전달받은 Object와 동일한지를 검증하는 것이다.

그리고 그 데이터는 'node' 라는 임시 변수에 옮겨서, node.next가 null이 아닐때까지 순차적으로 하나씩 검증하는 방식으로 진행했다.

그러면 교재의 솔루션 코드는 어떻게 구현되었을까?

~~~java
    @Override
	public int indexOf(Object target) {
		Node node = head;
		for (int i=0; i<size; i++) {
			if (equals(target, node.data)) {
				return i;
			}
			node = node.next;
		}
		return -1;
	}
~~~

다행히(?)거의 유사한 방식으로 해결했다. 다른 부분이라면 이터레이션을 제한하는 것을 나는 node라는 객체가 null일 때까지, 교재의 솔루션 코드는 size 라는 값으로 제한을 주었다.

사실 size라는 리스트를 크기를 관리하는 멤버변수가 있기 때문에 더욱 명확하게 카운팅하는 것을 표기할 수 있을 것인데, 나는 `LinkedList`의 컨셉이 객체의 주소값을 연결한다는 것에 초점을 맞추다 보니 객체의 null 체크를 했다.

그런데 문제는 `equals(x, y)` 메서드이다. 못보던 메서드가 사용되었다.

LinekdList 클래스 안에 추가된 메서드 인데, 로직은 다음과 같다.

~~~java
    private boolean equals(Object target, Object element) {
		if (target == null) {
			return element == null;
		} else {
			return target.equals(element);
		}
	}
~~~

이 역할은 `target` 이라는 Object의 null처리를 도와주는 `헬퍼 메서드` 이다.

private하게 제공된 헬퍼메서드의 역할은, 배열의 요소와 대상값이 같을 경우 true를 반환해주며, null 을 정상적으로 처리하는 프로그램의 안전성을 보장해준다.

### 알고리즘
{: .no_toc}

이 알고리즘은 리스트 전체의 값을 순차적으로 탐색해야 하기 때문에 **선형 알고리즘 O(n)**에 속한다.