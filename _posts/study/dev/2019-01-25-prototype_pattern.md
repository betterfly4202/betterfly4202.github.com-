---
title: "[디자인 패턴] Prototype pattern"
categories: "JAVA"
tags:
  - design pattern
  - prototype
toc: true
header:
  overlay_image: /assets/images/header/pattern.jpg
  overlay_filter: 0.5
  show_overlay_excerpt: false
---

# Prototype Pattern

아직 개발에 쓰이는 키워드들에 혼동되는 것들이 많아, 단어의 본질적인 의미가 떠오르지 않을때

그 단어의 이미지나, 어원 또는 한자어를 찾아보는 편이다.

사실 '프로토타입'이라는 말은 너무나 흔하고 일반적으로 많이 사용되는 단어이다.

'프로토타입'이라고 하면 **초기 단계** 정도의 개념이 떠오른다.

사전에 보면 프로토타입을 우리말로 '원형' 이라고 하는데,

이는 '原(근원 원)型(모형 형)'의 한자어이다.
다시 말해 **기본(근원)적인 모형** 으로 이해하면 될 것 같다.

결과적으로, 프로토타입이라는 패턴의 컨셉은 '기본적인 모형'을 이용하는 패턴 정도로 이해하면 될 것 같다.

---

## Prototype Pattern Example

~~~java
public class Book {

    private int bookdId;
    private String bookName;

    @Override
    public String toString() {
        return "Books{" +
                "bookdId=" + bookdId +
                ", bookName='" + bookName + '\'' +
                '}';
    }

    public String getBookName() {
        return bookName;
    }

    public void setBookName(String bookName) {
        this.bookName = bookName;
    }

    public int getBookdId() {
        return bookdId;
    }

    public void setBookdId(int bookdId) {
        this.bookdId = bookdId;
    }
}


public class BookShop implements Cloneable{
    private String shopName;
    List<Book> books = new ArrayList<>();

    @Override
    public String toString() {
        return "BookShop{" +
                "shopName='" + shopName + '\'' +
                ", books=" + books +
                '}';
    }

    public String getShopName() {
        return shopName;
    }

    public void setShopName(String shopName) {
        this.shopName = shopName;
    }

    public List<Book> getBooks() {
        return books;
    }

    public void setBooks(List<Book> books) {
        this.books = books;
    }


    public void loadData(){
        for (int i=0; i<10; i++){
            Book b = new Book();
            b.setBookdId(i);
            b.setBookName("Books "+i);
            getBooks().add(b);
        }
    }
    @Override
    protected Object clone() throws CloneNotSupportedException{
         return super.clone();
    }
}

public class Demo{
  public static void main(String[] args) throws CloneNotSupportedException {
        BookShop bs = new BookShop();
        bs.setShopName("Books Shop 1");
        bs.loadData();

        BookShop bs2 = (BookShop)bs.clone(); // 기존 clone() 메서드는 Object를 리턴하기 때문에 BookShop으로 캐스팅해줘야한다.
        bs2.setShopName("Books Shop 2");

        System.out.println(bs);
        System.out.println(bs2);
    }
}
~~~

프로토타입패턴을 이용하여 간단한 서점(BookShop)을 구현했다.

이 소스코드의 의도는 비슷한 구조의 여러 서점(BookShop Class)이 있다고 할때,

이 서점은 기본적으로 모두 동일한 책(Book Class)을 보유하고 있는 것으로 시작한다.

그렇기 때문에 BookShop 클래스 에서는 Cloneable을 구현하여 clone() 메서드를 오버라이드 했다.

(BookShop 클래스에서 loadData() 메서드는 10권의 책을 만들기 위함이다.)

그리고 Demo 클래스에서 'Books Shop 1' 라는 서점을 만들어서 10권의 책을 만들었다.

그리고  Books Shop 1 서점을 복사하여 'Books Shop 2' 라는 두번째 서점을 만들었다.

최종적으로 출력하면 다음과 같은 결과가 나온다.

![bookShop](/assets/images/study/dev/2019/1_bookshop.png)

Books Shop 1 서점의 이름만 바꾸고 책 리스트 내용은 그대로 복사하여 생성했기 때문에 내용이 그대로 복사된 모습이다.

이것이 기본적인 프로토타입 패턴의 컨셉이다.

여기까지 이해했다면, 기본적인 컨셉은 이해한 것이다.

**그러면 프로토타입은 왜 또는 어떠한 상황에서 사용(응용)될 수 있을까.**

## Prototype Pattern의 특징

## WHY?
{: .no_toc}
**GoF의 디자인 패턴** 책에서 프로토타입의 활용성을 다음과 같이 정의한다.
- 인스턴스화할 클래스를 런타임에 지정할 때 (동적로딩)
- 제품 클래스 계통과 병렬적으로 만드는 팩토리 클래스를 피하고 싶을 때
- 클래스의 인스턴스들이 서로 다른 상태 조합 중에 어느 하나일 때

말이 조금 어렵다.

하나씩 이해하기 쉽게 풀어서 생각해보자.

#### Runtime시 새로운 객체를 추가할 수 있다.
{: .no_toc}

>Run-time 이란?<br/>
**프로세스 또는 어플리케이션이 동작 중인 상태**를 말한다. <br/><br/>
이와달리 런타임이 실행되기전 상태를 'Compile-time'이라고 한다. <br/>

Run-time시 동적으로 컨트롤이 가능하다는 것은, 인스턴스를 매번 새로 생성하는 것이 아닌, 생성된 인스턴스를 복제하여 활용할 수 있다는 것이다.

위에 예시로 했던 서점을 생각해보자.

*만약 3개의 서점을 생성하는데, 이 3개의 서점이 모두 동일한 책을 보유하고 있다면?*

일반적으로 다음과 같이 구현할 것이다.

~~~java
public class Demo{
  public static void main(String[] args) throws CloneNotSupportedException {
        BookShop bs1 = new BookShop();
        bs1.setShopName("서점 1");
        bs1.loadData();
        
        BookShop bs2 = new BookShop();
        bs2.setShopName("서점 2");
        bs2.loadData();
        
        BookShop bs3 = new BookShop();
        bs3.setShopName("서점 3");
        bs3.loadData();
    }
} 
~~~

3개의 각기 다른 인스턴스를 만들어 각각의 서점의 이름과 동일한 도서 리스트를 생성해주는 모습이다.

하지만 이것은 불필요한 인스턴스 생성과 같은 작업을 3번이나 반복하고 있다!

프로토타입 패턴을 이용해서 다음과 같이 리팩토링할 수 있다.

~~~java
public class Demo{
  public static void main(String[] args) throws CloneNotSupportedException {
        BookShop bs1 = new BookShop();
        bs1.setShopName("서점 1");
        bs1.loadData();
        
        BookShop bs2 = (BookShop)bs1.clone();
        bs2.setShopName("서점 2");
        
        BookShop bs3 = (BookShop)bs1.clone();
        bs3.setShopName("서점 3");
    }
} 
~~~

이와 같이 동적으로 유연한 방법으로 새로운 인스턴스를 생성할 필요 없이 객체를 합성하여 새로운 행동으로 정의할 수 있다.

#### 구조를 다양화 함으로써 새로운 객체를 명세할 수 있다.
{: .no_toc}
많은 응용프로그램은 구성요소와 부분 구성요소의 복합을 통해 객체를 구축합니다. 예를 들어, 회로설계를 위한 편집기는 세부 회로를 모아서 큰 회로를 만듭니다. 이런 응용프로그램에서는 편의를 위한 복잡한 사용자 정의 구조를 사용자가 인스턴스화 하여 그 상황에 맞는 세부 회로를 계속 이용할 수 있도록 배려해 줄 때가 많습니다. 복합 회로 객체가 Clone() 연산을 구현함으로써 다른 구조를 갖는 회로의 기본 골격을 만듭니다.

#### 서브클래스의 수를 줄일수 있다.
{: .no_toc}
팩토리 메서드 패턴과 같이 상속 계층을 통해 클래스를 구현하는 것이 아닌, 원형 클래스를 복제하여 사용하기 때문에 상속하여 구현하는(서브 클래스) 사용을 줄일 수 있다.

---

프로토타입패턴에 대해서 기본적인 사용법과 특징을 알아보았다.

다음으로 프로토타입을 응용하여 사용하기 위해 필요한 **얕은복사(Shallow Copy)와 깊은복사(Deep Copy)**를 알아보자.


# [참고]
- https://www.holaxprogramming.com/2014/02/12/java-list-interface/
- https://leetaehoon.tistory.com/55
- https://donxu.tistory.com/entry/Prototype-pattern-%ED%94%84%EB%A1%9C%ED%86%A0%ED%83%80%EC%9E%85-%ED%8C%A8%ED%84%B4
- https://enter.tistory.com/108
- https://springframework.guru/gang-of-four-design-patterns/prototype-pattern/
