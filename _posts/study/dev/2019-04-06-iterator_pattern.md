---
title: "[디자인 패턴] Iterator Pattern"
categories: "JAVA"
tags:
  - design pattern
  - iterator
toc: true
header:
  overlay_image: /assets/images/header/pattern.jpg
  overlay_filter: 0.5
  show_overlay_excerpt: false
---

# Iterator Pattern

> 핵심 : `표현부`를 노출하지 않고, <span style="color:blue">집합 객체의 원소</span>들을 순차적으로 접근한다.

핵심은 사용자 입장에서 표현부를 노출하지 않고, 집합 객체의 접근 방법을 통일하여 순차적으로 접근할 수 있도록 도와준다는 것이다.

아주 간단한 예로 다음과 같이 `ArrayList` 자료구조를 사용하고 있는 객체가 있다고 가정하자.

~~~java
  List<String> arr = new ArrayList();
  arr.add("isFrist");
  arr.add("isSecond");
  arr.add("isThird");
~~~

이 객체의 원소들은 다음과 같이 접근할 수 있다.

~~~java
  for(int i=0; i<arr.size(); i++){
    arr.get(i);
  }
~~~

이 밖에도 다양한 루프를 통해 접근할 수 있지만, 어떻게 반복하여 접근하는지는 중요하지 않다.

중요한 것은 `arr.get(i)` 이 부분이다.

만약 이 자료구조가 `List`가 아닌, `Queue`로 바뀐다면 어떨까?

~~~java
  Queue<String> queue = new ConcurrentLinkedDeque<>();
  queue.offer("isFirst");
  queue.offer("isSecond");
  queue.offer("isThird");
~~~

Queue로 구현되었기 때문에 생성 과정도 변경되었지만, 이를 꺼내기 위해선 get()이 아닌, `peek()`또는 `poll()`로 꺼내야 한다.

이렇듯 자료구조에 따라 사용자가 객체의 원소에 접곤하는데 있어서, 매번 구현된 **자료구조에 따른 사용법과 구조를 파악하고 있어야만 접근이 가능**하다.

바로 이러한 불편과 데이터를 설계할시 다양한 커버리지를 위해서 객체 자료구조는 유동적으로 설계하며, 사용자는 `Iterator` 라는 인터페이스를 통해서 구현이나 반복 기법에 종속되지 않고 접근할 수 있게 된다.

---

## UML

![](/assets/images/study/dev/2019/4_Iterator-Pattern-UML.gif)

UML Diagram을 살펴보면, `Aggregate`와 `Iterator`라는 인터페이스를 통해 전체 구조를 설계한다.

궁극적으로 사용자는 `Iterator` 인터페이스에 종속되어 있는 메서드를 통해 접근할 수 있게 된다.

- **Aggregate** : Iterator 객체를 생성하는 인터페이스
- **ConcreateAggregate** : 해당하는 ConcreateIterator의 인스턴스를 반환하는 Iterator 생성 인터페이스
- **Iterator** : 원소를 접근하고 순회하는데 필요한 인터페이스
- **ConcreateIterator** : Iterator에 정의된 인터페이스를 구현하는 클래스로, 순회 과정 중 집합 객체 내에서 현재 위치를 기억

---

## 적용하기

사실 UML이 눈에 익지 않아, 그림만 보면 구조가 쉽게 파악되지 않는다.

간단한 예를 통해 Iterator 패턴의 사용 과정을 살펴보자.

> **설정** : 새로운 웹 프로젝트의 TF조직에 개발자 3명이 투입 되었다.<br/>  각각의 역할은 다음과 같다. <br/>
> - Model : 데이터베이스에서 데이터를 추출하여 `Controller`에게 전달
> - Controller : `Model`에게 전달 받은 데이터를 가공하여 `View`에 전달
> - View :  `Controller`에게 전달 받은 데이터를 사용자(클라이언트)에게 노출<br/>
> 
> 나는 이 중 `Controller` 역할을 담당하게 되었다. <br/><br/> 
> 내가 할 일은 `Model`로 부터 받은 데이터를 잘 가공하여, `View`에게 전달해야한다. <br/>
> 하지만 아직 `Model`담당자가 어떻게 데이터를 전달해 줄 것인지 API 명세가 정리되지 않았다. <br/> 
> 시간이 촉박하여 가상 객체를 통해서라도 어플리케이션 로직을 설계해야 한다.
> **어떻게 해야 할까?**

어떻게 하긴, 우린 지금 `Iterator 패턴`을 학습 중이다.

Iterator 패턴을 통해서 이 문제를 해결할 수 있다.

간단한 코드와 함께 확인해보자.

### Aggregate

Iterator 패턴에서는 `Aggregate` 를 통해 **iterator** 객체를 구현하고자 한다.

여기서 말하는 `iterator`는 **표현부(클라이언트)**에서 접근을 위한 메서드 이다.

~~~java
public interface Aggregate {
    Iterator iterator();
}
~~~

### ConcreateAggregate

**ConcreateAggregate** 객체는 Aggregate의 iterator() 구현부 이다.

이 구현부에서 사실상 어떤 컬렉션(자료구조)를 사용할지가 정의된다.

현재는 `ArrayList`로 설계했지만, 이 부분의 어떤 컬렉션을 생성하느냐에 따라 구현부를 설계하면 된다.

~~~java
public class Company implements Aggregate{
    private List<Employee> employees;

    public Company(){
        this.employees = new ArrayList<>();
    }

    public Employee getEmployee(int index){
        return this.employees.get(index);
    }

    public void appendEmployee(Employee employee){
        this.employees.add(employee);
    }

    public int employeesSize(){
        return this.employees.size();
    }
    @Override
    public Iterator iterator() {
        return new ListIterator<Employee>(this.employees);
    }
}
~~~

다음은 위 `Company`객체에서 사용된 *Employee* 객체의 내용이다.

~~~java
public class Employee {
    private String name;
    private int age;

    public Employee(String name, int age){
        this.name = name;
        this.age = age;
    }

    public String getName(){
        return this.name;
    }

    public int getAge(){
        return this.age;
    }
}
~~~

### Iterator

원소의 접근을 순회하기 위한 인터페이스를 제공한다.

순회하기 위해서 어떤 옵션을 추가할 것인지 해당 인터페이스를 통해 정의하면 된다.

~~~java
public interface Iterator<T> {
    boolean hasNext();
    T next();
    T currentItem();
}
~~~

### ConcreateIterator

Iterator 인터페이스의 구현부 이다.

순회를 위해 필요한 비지니스로직을 구현한다.

~~~java
public class ListIterator<T> implements Iterator{

    private List<T> list;
    private int size;
    private int currentIndex;

    public ListIterator(List<T> list){
        this.list = list;
        this.size = this.list.size();
        this.currentIndex = 0;
    }

    @Override
    public boolean hasNext() {
        return currentIndex < size;
    }

    @Override
    public T next() {
        T item = this.list.get(this.currentIndex);
        this.currentIndex++;

        return item;
    }

    @Override
    public Object currentItem() {
        return this.list.get(this.currentIndex);
    }
}
~~~

위 구조를 UML로 나타내면 다음과 같다.

![](/assets/images/study/dev/2019/4_iterator_pattern_uml.png)

---

## 사용하기

설계한 Iterator 패턴을 사용해보자.

~~~java
public class IteratorTest {
    private Company company;

    @Before
    public void setUp(){
        company = new Company();
        company.appendEmployee(new Employee("자바맨", 95));
        company.appendEmployee(new Employee("betterFLY", 14));
        company.appendEmployee(new Employee("슬립친구", 35));
    }

    @Test
    public void 반복자_패턴(){
        Iterator<Employee> iterator = company.iterator();

        assertThat(iterator.currentItem().getName(), is("자바맨"));
        assertThat(iterator.currentItem().getAge(), is(95));
        assertThat(company.getEmployee(2).getName(), is("슬립친구"));
        assertThat(company.employeesSize(), is(3));

        while(iterator.hasNext()){
            Employee employee = iterator.next();
            System.out.println(employee.getName());
        }
    }
}
~~~

iterator를 접근하기위하여 `while`루프를 통해 손쉽게 접근이 가능하다.

~~~java
  while(iterator.hasNext()){
      Employee employee = iterator.next();
      System.out.println(employee.getName());
  }
~~~

구현부에서 어떤 컬렉션이 오느냐에 상관없이 표현부에서는 위와같은 방식으로 처리할 수 있기 떄문에,

서두에 얘기했듯이 정의되지 않은 자료구조에서 보다 커버리지가 높은 코드를 작성할 수 있게 된다.