---
title: "Call by reference in Java"
categories: "JAVA"
tags:
  - Call by reference
  - Call-by-{Value | Reference} 
---


포스팅에 앞서 오늘도 역시나 무지함을 깨닫고 시작한다.

평소에도 기초가 부족함을 많이 느꼈지만, 최근 인스턴스 또는 객체 간 접근의 흐름이 헷갈리면서 call by reference / call by value 라는 개념을 확실히 익히기 위해서 업무 중에 기록해 두고 여유가 있을때 포스팅 하기로 했다.

자바에서 이 두 매커니즘이 어떻게 적용되는지 조사를 시작했다.

call by value와 call by reference 를 검색하면 가장 많이 볼 수 있는 C기반의 swap 함수로 그 개념을 이해할 수 있다.


```cfml 
    void valSwap(int a, int b){
        int temp = a;
        a = b;
        b = temp;
    }
    
    int main(int args, char** argv){
        int a = 3;
        int b = 5;
        valSwap(a,b);
        cout << "a: " << a << "|| b: " << b << endl;
    } 

```

>결과 :  a: 10 || b:20


너무나 당연한 결과이지만 valSwap함수를 호출했지만 스왑이 이루어지지 않는다. 이를 stack 그림으로 참고해 보면 이해가 쉽다.


![call_by_value](/assets/images/study/dev/2018/5_call%20by%20value.png)

swap 이라는 함수는 호출되지만 각 변수(a,b,temp)가 할당하는 메모리 주소는 공유되지 않는다.

때문에 main함수에서 선언한 argument a와 b의 값에 영향을 주지 않는다.


---
#### argument vs parameter

우리는 흔히 argument와 parameter를 혼용해서 쓰기도한다.
하지만 엄밀히 따지면 두 단어가 갖는 의미는 다르다.

두 단어 모두 인자를 표현하는 단어지만 구분해서 쓰자면 **formal-parameter(형식인자)** 와 **actual-parameter(실인자)** 로 이해하는 것이 적당하다. <br/>
다음 코드를 통해 그 차이를 살펴보자.
 
 ```java
 public class ArgumentAndParameter{
    public void runFunc(String param) {
         System.out.println("this is " + param);
     }
     public static void main(String[] value) {
         String args = "I am Arguments";
         runFunc(args);
     }
 }

```

위 샘플코드에서 runFunc(String param) 함수에 선언된 변수 param은 **formal-parameter(형식인자)** 즉, Parameter이다. (우리 말로 '매개변수' 라고 표현한다.)

그리고 main함수에서 선언한 변수 args는 **actual-parameter(실인자)** 즉, Argument이다. (우리 말로 '인자' 로 표현한다)

---

다시 본론으로 돌아와 call by reference 를 위의 swap 함수를 통해 살펴보자.



```cfml
 
    void swap(int *a, int *b)
    {
        int temp = *a;
        *a = *b;
        *b = temp;
    }
    
    int main(int args, char** argv){
        int a = 10;
        int b = 20;
        swap(&a, &b);
        cout << "a: " << a << "|| b: " << b << endl;
    }
    

```

>결과 : a: 20 || b:10

이전과 다른 것은 포인터를 통해 변수에 접근했다. 그 결과 스왑이 정상적으로 이루어졌다.

어떻게 이런 결과가 나왔는지 다음의 그림을 참고해보자. 

![call_by_ref](/assets/images/study/dev/2018/5_call%20by%20ref.png)

위 그림에서 눈여겨 볼 것은 1번 변수 a,b가 생성되며 표기된 좌측의 **100**, **104** 라는 메모리 주소이다.

call by value 의 예제처럼 일반적으로 Parameter(매개변수)를 선언하여 사용한다면, 각 매개변수는 저마다 다른 메모리를 참조한다.

하지만 포인터를 통해 생성된 매개변수는 그림과 같이 변수명이 같은 메모리를 참조하게 되어 결과적으로 변수 a, b의 결과가 스와핑되는 것을 볼 수 있다.

---

여기까지가 일반적으로 알고 있는 C를 통한 call by value 와 call by reference의 차이점이었다.

이렇게 끝나면 참 평화롭겠지만, 자바는 조금 더 설명이 필요하다.

### Call by value in Java

![datatype_in_java](/assets/images/study/dev/2018/5_DataType_java.png)
<figcaption class="caption">Data type in Java</figcaption>

그림에서 보이는 좌측의 Primitive 타입(원시타입)만이 소위 call by value 라는 매커니즘이 적용된다. 


 ```java
 public class CallByInJava{
    public static void runValue(){
         int a = 1;
         int b = a;
         b = 2;
         System.out.println("runValue, "+a); 
     }
     
     
     public static void main(String [] args){
        runValue();
     }
 }

```
> 결과 : runValue, 1 

위 예제의 결과 a는 당연히 '1'이 출력된다. 너무 당연해서 설명을 하기도 어렵지만, runValue에서 변수 b는 변수 a를 복제한 것이고, 복제한 b 의 값만 변경했기 때문에 a의 결과에 영향을 주지 않는다.



 ```java
 public class RefClass{
    public int id;
    RefClass(int id){
        this.id = id;
    }
 }
 
 public class CallByInJava{
    public static void runValue(){
         int a = 1;
         int b = a;
         b = 2;
         System.out.println("runValue, "+a); 
     }
     
     public static void runRef(){
        RefClass ref = new RefClass(5);
        RefClass subRef = ref;
        subRef.id = 10;
        System.out.println("runRef, "+ref.id);
     }
     
     public static void main(String [] args){
         runValue();
         runRef();
      }
 }

```

> 결과 : runValue, 1 , runRef, 10 

subRef.id를 변경하였는데 ref.id의 결과가 바뀌었다. 여기서 우리는 '참조(Reference)'라는 개념을 이해해야 한다.

이는 앞서 살펴보았던 C 의 포인터를 통해 메모리 주소를 가리키는 것과 유사한 개념이다.

흔히 자바에서 new 를 통해 생성되는 객체를 '인스턴스'라고 표현한다. 이 모든 인스턴스들은 저마다 메모리를 참조하고 있다.

따라서 위 예제와 같이 **RefClass ref=new RefClass(5)** 를 통해 생성된 RefClass 의 인스턴스는 특정한 메모리를 참조한다.  

그리고 새로운 변수 subRef 는 ref 를 참조한다.

<pre>
    RefClass subRef = ref;
</pre<

이것은 ref 와 subRef 두 변수가 모두 같은 인스턴스를 **참조** 하고 있다는 것이다.

따라서 subRef를 변경했지만 ref의 id값이 변경되는 것이다.

이처럼 자바의 데이터타입을 보여주는 위 그림 중 Primitive 타입을 제외한 모든 경우 Call by reference 방식을 통해 인자가 전달된다.


---

- [이미지 참고](http://choieun.tistory.com/entry/Call-by-value%EC%99%80-Call-by-reference) 
- [내용 참고_머루의 개발 블로그](http://wonwoo.ml/index.php/post/1679)
- [내용 참고_Mussebio's HuRrah Blog](http://mussebio.blogspot.kr/2012/05/java-call-by-valuereference.html)
- [내용 참고_생활코딩 강좌](https://opentutorials.org/course/1223/5375)  