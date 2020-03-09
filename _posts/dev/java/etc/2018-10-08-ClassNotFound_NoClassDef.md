---
title: "ClassNotFoundException && NoClassDefFoundError"
categories: "Java"
tags:
  - ClassNotFoundException
  - NoClassDefFoundError
---


ClassNotFoundException 그리고 NoClassDefFoundError

가장 쉽게 접하는 예외상황이기도 하고, 가장 단순한 문제이기도 하다.

그런데 문득 두 다 비슷한 문제로 보이는데 무슨 차이가 있는지 궁금해서 찾아보았다.

그러던 중 NoClassDefFoundError 발생하는 과정은 생소한 부분이 있어 기록하려한다.

---

### ClassNotFoundException
: 클래스 로드 시 지정된 패스에 해당 클래스를 찾을 수 없는 경우


```java
public class TargetClass {
    public void execute() throws ClassNotFoundException{
        Class c = Class.forName("web.ClassNotFoundTestt");
        System.out.println("terminated!");
    }


    public static void main(String[] args) {
        try {
            new TargetClass().execute();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

```

![ClassNotFound](/assets/images/study/dev/2018/10_classnotfound.png)

위와 같이 web패키지아래 ClassNotFoundTestt 라는 클래스를 로딩하지만 해당 클래스가 존재하지 않는 경우이다.

너무나 직관적이고 설명이 따로 필요 없을 정도로 간단하다.

위 예외 상황의 경우

at web.TargetClass.execute(TargetClass.java:10)

TargetClass의 10번째 라인에서 참조하고있는 web.ClassNotFoundTestt 라는 클래스가 존재하는지 확인해보면된다.

일반적으로 우리는 아주 성능좋은 IDE를 사용하기 때문에 컴파일 시 이렇게 참조된 클래스가 존재하지 않는 경우 아주 친절하게 참조하고 있는 클래스 패스 또는 클래스 정보의 문제가 있음을 알기 좋게 표현해준다.

하지만 문제는 NoClassDefFoundError 이다.

다음 예외는 비슷해보이지만 다르다.

---

### NoClassDefFoundError
: 클래스가 Java 컴파일러 단계에서 생성되었지만 런타임시 종속된 클래스가 발견되지 않는 경우

말이 어렵다.

컴파일러 단계, 런타임, 종속된 클래스...

글로 이해하려 하면 이해가 되지 않는다 직접 보면서 확인해보자.

쉬운 예시로 직접 생성해가며 진행해보는게 이해가 빠르다.

(Mac의 터미널 환경에서 테스트를 진행했다. 자바 버전 1.8)

> 테스트를 하며 정말 큰 삽질을 해서 애를 많이 먹었는데, IDE를 통해 컴파일을 하면 이와 같이 테스트를 진행할 수 없다. IDE툴 마다 정확한 컴파일 원리는 파악하지 못했지만, 인텔리제이의 경우 파일 수정 후 저장을 할 경우 자동으로 컴파일 되며, 컴파일 된 파일은 따로 공간에 저장되는 것 같다.

테스트를 위해 다음과 같이 클래스를 만들었다.

~~~java

class NoClassDefErrorMain{
	public static void main(String [] args){
		Target target = new Target();
		target.print();
	}
}

class Target extends TargetMain{

}


class TargetMain{
	public void print(){
		System.out.println("call target main class!!!");
	}
}

~~~

각각 3개의 서로 다른 java(class)파일을 만들었다.

그 후 메인메서드를 갖고 있는 NoClassDefErrorMain 클래스만 컴파일 해주면 해당 클래스에 포함되어 있는 모든 자바파일들이 컴파일된다.

> javac NoClassDefErrorMain.java

컴파일하면 다음과 같은 결과를 확인할 수 있다.

![compile_result](/assets/images/study/dev/2018/10_compile_result.png)

NoClassDefErrorMain 클래스에 종속되어 있는 모든 자바 파일들이 컴파일되어 .class 파일로 떨어졌다.

이제 모든 컴파일 작업이 완료되었으니 실행해준다.

> java NoClassDefErrorMain

![NoClassDef_run](/assets/images/study/dev/2018/10_NoClassDef_run.png)

사진과 같이 정상적으로 실행되어 *call target main class!!!* 를 확인할 수 있다.

여기까지가 정상적으로 컴파일 된 자바파일을 실행한 결과이다.

다시 처음으로 돌아가, NoClassDefFoundError 에러의 정의를 되짚어 보면

> 클래스가 Java 컴파일러 단계에서 생성되었지만 런타임시 종속된 클래스가 발견되지 않는 경우

라고 한다. 그러면 우리는 지금 "**Java 컴파일러에서 단계에서 생성**" 까지 진행한 것이다.

그 다음 스탭인 "**런타임시 종속된 클래스가 발견되지 않는 경우**" 를 진행하기 위하여, 컴파일하여 만들어진 종속된 클래스

*TargetMain.class* 파일을 삭제 한다.

> rm TargetMain.class

![delete_targetMain](/assets/images/study/dev/2018/10_delete_TargetMain.png)

절차대로 진행했다면 위와 같이 파일의 결과가 나온다.

그러면 이제 다시 메인 메서드를 호출하자.

> java NoClassDefErrorMain

![noDefClass](/assets/images/study/dev/2018/10_NoClassDefFound.png)

드디어 나왔다. NoClassDefFoundError 이다.

앞서 진행한대로, 컴파일하는 단계에선 존재했지만 실제로 런타임하는 과정에서 참조하고 있는 클래스가 없는 경우 이와 같은 에러를 발생시킨다.

(현재 구조에서 Target.class 파일을 삭제 후 실행하는 경우 ClassNotFoundException 이 발생한다.)

위의 예시를 통해 정리하자면 이렇다.

**ClassNotFoundException**

JVM의 Class Loader가 NoClassDefErrorMain클래스에서 Target클래스를 로딩하려고 할 때 해당 클래스(Target)찾을 수 없는 경우

**NoClassDefFoundError**

JVM의 Class Loader가 Target클래스를 로딩 하는 중 TargetMain클래스를 내재적으로 로딩하던 중 TargetMain 클래스를 찾을 수 없는 경우.

다시 말해, Target클래스 내부에 정의된 TargetMain클래스가 class path에 존재하지 않아서 정상적으로 실행할 수 없는 경우를 말한다.

---

JVM의 구동 원리를 이전에 다루었던 적이 있었는데, 눈으로만 글을 봐서 피부로 와닿는 느낌이 없었는데 이번 이슈를 정리하며 조금은 JVM과 친해진 것 같다.

다음엔 인텔리제이가 어떤식으로 컴파일을 하고 구동되는지를 분석해보는 시간을 갖는 것이 좋을 것 같다.