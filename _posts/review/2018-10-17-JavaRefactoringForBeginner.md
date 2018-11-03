---
title: "자바로 배우는 리팩토링 입문(Java Refactoring For Beginner)"
layout: post
date: 2018-10-17 02:03
image: 
headerImage: true
tag: 
- 리팩토링
category: review
author: betterFLY
description:
---

### 자바로 배우는 리팩토링 입문(Java Refactoring For Beginner)

그동안 읽었던 책을 정리하고 싶었는데, 사실 잘 기억이 나질 않는다.

그보다 지금 읽는 것이라도 정리를 해 나가는게 좋을 것 같아 차근차근 업데이트해보려한다.

리팩토링은 개발자가 늘상 고민해야하는 과제이다.

변수명 하나를 바꾸는 것도 리팩토링 과정일 것이며, 보다 나은 프로그래밍을 고민한다면 늘상 고민하고 해결해나가야 하는 과정이다.

리팩토링 관련해서는 거의 바이블과 같은 책이 있지만

![리팩토링](/assets/images/181017/book_refactoring.jpg)
<figcaption class="caption">리팩토링(마틴 파울러)</figcaption>

위 책은 조금 어렵다.

그러던 중 스터디원이 자바로 리팩토링하는 과정을 쉽게 정리한 책을 추천해주었다.

전체 15개의 챕터로 구성되어있는데, 내용이 잘 정리되었고 예제가 잘 정리되어 있어서 읽기에도 좋았고

역시 이런 기술적인 서적은 금방 잊어먹기 때문에 간단히 기록해 두려한다.

> 2018.10.17

#### 1장. 매직 넘버를 기호 상수로 치환

> *매직넘버*란? <br/> 프로그래밍에서 말하는 매직넘버는 특정한 조건 혹은 필드 등의 값을 숫자로 표현하는 것을 말한다.

예를들어 다음과 같은 코드가 있다면,

~~~java
    if(100 < input.length){
        ...
    }
~~~

위 상황의 경우 내용은 직관적으로 쉽지만 '100'이라는 조건이 어떤 값을 의미하는지 파악하기 어렵다.

하지만 다음과 같이 정의해보자.

~~~java
public class Something{
    public static final int MAX_INPUT_LENGTH = 100;
}

if(Something.MAX_INPUT_LENGTH < input.length){
    ...
}
~~~

이처럼 상수로 치환하여 사용한다면, 같은 '100' 이라는 값이 정확히 어떤 의미로 사용하는지 이해하기 쉽다.

단순히 직관적으로 값의 의미를 이해하는 것 뿐만 아니다.

다음의 예제를 보자.

~~~java
public class Sample1{
    public void sample(){
        if(100 < input.length){
            ...
        }
    }
}


public class Sample2{
    public void sample(){
        if(100 < input.length){
            ...
        }
    }
}

public class Sampl31{
    public void sample(){
        if(100 < input.length){
            ...
        }
    }
}
...
~~~

이처럼 각각의 클래스에서 input.length를 검증하는 로직을 사용하는데 기준 값이 100이 아니라 200으로 변경된다면...

모든 클래스의 '100' 값을 '200'으로 바꿔야 한다.

하지만 '기호 상수(symbolic constant)'로 처리한다면, 해당 상수의 값만 바꿔주면 한번에 해결할 수 있다.

최근 작업했던 어떤 프로젝트에서 이처럼 무분별한 매직넘버가 남용된 상황을 본 적이 있다.

알 수 없이 중구난방으로 사용되고 있는 숫자값때문에 정말 크게 애를 먹었던 기억이 난다.

가장 첫 장에 소개된 만큼 간단하지만 아주 중요한 작업이다.

프로그래밍은 혼자하는 것이 아니기 떄문이다.


> 2018.10.20

#### 2장. 제어 플래그 삭제

> *제어 플래그*란? <br/> 프로그래밍에서 '상태를 기록하고 처리 흐름을 제어하기 위한 boolean 타입의 변수'

<일반적인 제어 플래그 사용의 모습>

~~~java
boolean flag = true;
while(flag){
    ... 

    if(A){
        flag = false;
    }else{
        // do something
    }
}
~~~

*제어 플래그 삭제*

~~~java

while(flag){
    ... 

    if(A){
        break;
    }else{
        // do something
    }
}
~~~
