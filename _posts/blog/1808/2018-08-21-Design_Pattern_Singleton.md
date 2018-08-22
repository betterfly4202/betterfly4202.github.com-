---
title: "Design Pattern_Singleton Pattern"
layout: post
date: 2018-08-21 20:33
image: 
headerImage: true
tag:
- Design pattern
- singleton
category: blog
author: betterFLY
description:
sitemap :
  changefreq : daily
  priority : 1.0
---

## [Singleton Pattern]

![singleton](/assets/images/180823/Singleton-pattern-class-diagram.jpg)

싱글턴패턴은 이름 그대로 아주 심플하다.

인스턴스를 하나만 생성하여 설계하는 디자인 패턴이다.

그렇다면 왜, 어떤 상황에서 인스턴스를 하나만 생성하는 것이 유용할까?

흔히 connection pool, thread pool, device settings 과 같은 상황에서 싱글턴패턴은 유용하다.

위와 같은 환경에서 인스턴스를 하나만 만들어, 자원의 낭비를 방지하고 엄격하게 관리해야 할 설정들을 하나의 인스턴스로 관리함으로써 효과적인 관리를 할 수 있다.

>개발중인 시스템에서 스피커의 볼륨을 제어하는 클래스가 필요하다면?

만약 이런 클래스의 인스턴스를 100개 만들어 관리한다고 가정할때, 우리는 볼륨을 "1" 만큼 올리는데 100개의 모든 클래스를 찾아 볼륨을 올려줘야 한다.

이는 매우 불필요한 복잡도를 야기시키고, 시스템의 리소스만 잡아먹는 비효율적인 작업일 것이다.

그렇다면 이렇게 시스템의 스피커를 관리하는 클래스를 *싱글턴패턴*을 통해 하나의 인스턴스로 관리해보자.


~~~java

public class SystemSpeaker {
    // SystemSpeaker의 instance 라는 멤버변수는 유일한 값을 유지해야 하기 때문에 static 으로 생성 
    static private SystemSpeaker instance;
    private int volume;

    // Singleton 이기 때문에 외부에서 생성하지 못하도록 private 제한
    private SystemSpeaker(){
        volume = 5;
    }

    public static SystemSpeaker getInstance() {
        // 하나의 인스턴스만 존재해야 하기 때문에 instance에 null을 체크하여 하나의 시스템 스피커 인스턴스만을 생성함
        if(instance == null){
            instance = new SystemSpeaker();
            System.out.println("make new instance");
        }else{
            System.out.println("instance is already made");
        }
        return instance;
    }

    public int getVolume() {
        return volume;
    }

    public void setVolume(int volume) {
        this.volume = volume;
    }
}

~~~ 

위 클래스를 메인함수에서 호출한다면 아래와 같은 결과를 확인할 수 있다.

![speaker_result](/assets/images/180823/speaker_main.png)

동일한 메모리 주소를 바라보고 있는 하나의 인스턴스 이므로 당연히 동일한 결과를 확인할 수 있다.

---

### 여기까지는 Single Thread 상에선 문제가 되지 않을 것이다.

하지만 Multi Thread 환경에서는   


http://asfirstalways.tistory.com/335  