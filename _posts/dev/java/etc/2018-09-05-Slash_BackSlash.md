---
title: "Slash & Back Slash"
categories: "Java"
tags:
---

## Slash & Back Slash

평소 당연히 그렇게 사용해왔는데, 왜 그렇게 사용했는지에 대해선 생각해보지 않았던 것 같다.

특히 정규식을 사용할때 거의 필수적으로 사용했던 역슬래쉬(\\) 에 대한 내용이다.

[지난일기](https://betterfly88.github.io/slash/ "slash & back slash") 이전에 이미 크게 삽질을 한 후 스스로의 무지함에 놀라며 기록을 남겨야겠단 생각을 했다.

---

아주 기본적으로 우리는 개행문자를 파싱할때 다음과 같이 사용한다.

~~~java
  String message = "Welcome to\n betterFLY world!";
  System.out.println(message);
~~~

위 결과를 찍어보면
>Welcome to \n
>betterFLY world!

와 같이 결과가 나올것이다.

그리고 개행문자를 공백으로 제거한다면,

~~~java
  System.out.println(message.replaceAll("\n", ""));
~~~

>Welcome to betterFLY world!

위와 같은 결과를 받아볼 수 있다.

"\n" 는 그 자체가 개행문자를 표현한다고 생각했을 뿐, 각각의 역슬러쉬(\\)와 문자열(n)에 대해서 깊이 고민해보지 않았다.


그리고 다음과 같은 질답을 찾았다.

#### 질문
![slash_question](/assets/images/study/dev/2018/9_Question.png)

#### 답변
![slash_answer](/assets/images/study/dev/2018/9_Answer.png)

<figcaption class="caption">출처 : Okky 커뮤니티</figcaption>
---

그렇다.

역슬래쉬는 그 자체가 *이스케이프문자* 이므로, 그 자체가 어떤 의미가 있거나 표현할 수 없다.

자바를 비롯한 C계열의 대부분의 언어에서 역슬래쉬는 이스케이프 문자로 사용된다.

- Escape Literal 이란?
  - Escape : 달아나다
  - Literal : 프로그래밍 언어에서 Literal의 일반적인 의미는 해당 언어가 처리하는 실제 데이터를 의미한다.

  이 두 단어를 조합하면 '실제 데이터의 의미를 달아나다(벗어나다)' 정도로 이해할 수 있을 것 갘다.

  즉, 이스케이프 문자를 통해 그 다음에 오는 리터럴(문자)가 가지는 의미를 무시한다는 것이다.
  
그래서 이스케이프 일부 문자에 대해 단일 문자 이스케이프 시퀀스를 제공한다.

### Escape Sequence
![Escape_Sequence](/assets/images/study/dev/2018/9_escape_sequence.png)

