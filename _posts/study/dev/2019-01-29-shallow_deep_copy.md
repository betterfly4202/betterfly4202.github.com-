---
title: "[디자인 패턴] Prototype pattern (Shallow Copy & Deep Copy)"
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

앞서 [프로토타입 패턴](https://betterfly88.github.io/dev/prototype_pattern/)의 기본적인 사용법과 특징에 대해서 알아보았다.

하지만 프로토타입 패턴을 제대로 이해하고 사용하기 위해서 꼭 알아두어야 할 개념이 있다.

바로 **얕은복사(Shallow Copy)와 깊은복사(Deep Copy)**이다.

## 얕은복사와 깊은복사(Shallow Copy & Deep Copy)

소위 얕은(Shallow) 복사라하면, 객체의 주소값(?)만을 복사하여 객체를 복사하는 것을 말한다.
때문에 A라는 객체와 A를 복사한 A' 라는 2개의 객체가 존재할 때.
A' 객체의 어떤 값을 변화시키면, A객체도 같이 바뀌게 된다.

반면 깊은 복사(Deep copy)는 객체 주소값을 참조하는 것이 아니라, 객체의 값 그 자체를 복사한다.
때문에 복사한 객체 어느것의 값이 변하여도, 복사된 이후로는 서로에게 영향을 주지 않게 된다.

![image](https://springframework.guru/wp-content/uploads/2015/04/4-23-2015-1-55-29-AM-300x210.jpg)

텍스트를 보면 이해가 어렵다. 코드를 통해 확인해보자.

다시 위에 샘플코드를 생각해보자.

위 샘플코드에서는 각각의 서점에 모두 동일하게 책이 10권씩 생성되었다.

그런데 만약 어떤 서점에 1권의 책이 분실되었다면 어떻게 될까?

~~~java
public class Demo{
  public static void main(String[] args) throws CloneNotSupportedException {
        BookShop bs = new BookShop();
        bs.setShopName("Books Shop 1");
        bs.loadData();

        BookShop bs2 = (BookShop)bs.clone();
        bs2.setShopName("Books Shop 2");

        bs.getBooks().remove(1); // index 1번째 책 분실

        System.out.println(bs);
        System.out.println(bs2);
    }
}
~~~

bs(Books Shop 1)의 도서리스트(getBooks())의 1번째 index만 제거해주었다.

결과는 어떨까?

![rm_bookShop](/assets/images/study/dev/2019/1_bs_remove.png)

이미지와 같이 두 서점의 1번째 인덱스 값이 모두 삭제되었다.

눈치를 챘을지 모르지만 두 서점의 books(ArrayList) 다음의 해쉬코드값이 동일하다. (**books@1250923891**)

이것은 두 서점이 동일한 도서리스트(ArrayList)를 참조하고 있다는 것이다.

이것을 바로 <code>얕은복사(Shallow Copy)</code>라고 한다.

객체는 복사가 되었는데, 참조 주소값만 복사되었기 때문에 그 객체를 사용하는 모든 결과 값은 동일할 수 밖에 없다.

당연한 것이다.

예를들어, 내 통장과 연결된 체크카드를 우리 엄마와 공유한다고 가정하자

내 체크카드 번호는 11-22-3 이다.

우리엄마도 사용하기 위해선 체크카드가 한개 더 필요하다.

새로 발급 받은 카드의 연결된 계좌 역시 11-22-3 이다.

같은 시간에 나는 회사에서 커피를 한잔 사먹고, 엄마는 동네에서 식사를 하고 각자 보유하고 있는 카드로 결제를 한다면?

카드의 객체는 서로 다르지만, 카드가 갖고 있는 계좌(참조 주소값)는 동일하기 때문에 결국 내 통장에서 두 결제 금액이 모두 빠져나간다.

이것이 **'얕은복사(Shallow Copy)'**이다.

---

다시 서점의 예제로 돌아가서, 두 도서리스트를 따로 관리하고 싶다면 어떻게 할까?

다시 말해, 깊은복사(Deep Copy)를 하려면 어떻게 해야할까.

BookShop에서 기존에 클로닝(Cloning)하는 부분을 다시 보자.

~~~java
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
~~~

>super.clone()

이와 같이 복제를 하는 것은 해당 클래스를 복사하는 것이다.

따라서 해당 클래스에 종속되어 있는 원시(primitive)적인 필드거나, immutable한 객체라면 클래스를 복사하여 각각의 값들을 별도로 관리해줄 수 있다.

하지만 List, Map 혹은 mutable한 객체라면 깊은복사를 통한 핸들링이 필요하다.

위 경우 books라는 필드(ArrayList)가 바로 mutable한 객체이기 때문에 객체를 복사시(clone()) 이를 깊은복사가 적용될 수 있도록 구현해보자.

~~~java
public BookShop clone() {
    BookShop bs = new BookShop();
    for(Book b : this.getBooks()){
      bs.getBooks.add(b);
    }
    return bs;
}
~~~

바뀐 부분을 보면, 

우선 Cloneable의 clone()을 오버라이드 하는것이 아니기 때문에, <code>예외처리(CloneNotSupportedException)</code>를 제거 했다.

그리고 이 객체는 BookShop이기 때문에 리턴값을 변경했고.

원본 인스턴스에 보유하고 있는 도서 목록(getBooks())을 새로운 BookShop 클래스에 담아서 리턴해준다.

이로써 복제한 모든 BookShop 클래스는 원본의 도서 목록(getBooks())를 갖을 수 있게 된다.

결과를 보자.

![deep_copy](/assets/images/study/dev/2019/1_deep_copy.png)

> 포토로타입 패턴을 사용시 얕은 복사와 깊은 복사 중에 무엇을 사용해야할까?  <br/>
그것은 전적으로 요구사항에 달려있습니다. <br/>
만약 객체가 primitive 필드거나 immutable 객체라면(whose state cannot change, once crated; 생성되면 바꿀수 없는) 얕은 복사를 사용하세요. <br/>
객체가 다른 mutable 객체를 참조한다면, 그때 얕은복사와 깊은복사 중에 선택하십시오. <br/>
예를들어, 참조값이 바뀔일이 없다면, 깊은 카피를 피하고 얕은 카피를 선택하십시오. <br/>
그러나 참조값이 변하거나 수정값이 어플리케이션의 다른 행동에 영향을 준다면 깊은 복사를 해야 합니다.

---

다소 장황했지만, <code>Protype Pattern</code>에 대해서 알아보았다.

GoF의 디자인 패턴 책 말머리에도 언급되었지만, 모든 패턴을 완벽하게 이해하고 외우고 있을 필요도 없을 뿐만 아니라, 그럴 수도 없다.

각 패턴의 큰 개념만 제대로 이해하고 있고, 자신이 설계하는 프로젝트에서 유연하게 구현할 수 있으면 된다.

그런데 나는 아직 책을 한번 읽거나, 인터넷 어딘가에서 누가 정리한 글을 대충 읽어서는 절대 기억도 못하고, 응용하지 못한다.

그래서 내용을 제대로 이해하고 싶어 이렇게 장황하게 정리하게 되었다.

## 관련 패턴

<code>Abstract Factory</code>