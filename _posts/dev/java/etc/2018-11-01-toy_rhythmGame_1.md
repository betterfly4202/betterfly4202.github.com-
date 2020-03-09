---
title: "[음악게임 만들기] - 01.프로젝트 시작 및 Intro화면 만들기"
categories: "Java"
tags:
  - Swing
---


## 01. Java Swing을 이용한 음악게임 만들기

>  소스코드는 깃헙에서 관리한다. <br/>https://github.com/betterfly88/SenseOfRhythm

평소 즐겨보던 유튜버 '동빈나'님의 컨텐츠를 보던 중 재미있는 주제가 있어 공부할겸, 그리고 기록으로 남겨 보려 한다.

정말 오랜만에 Java의 Swing컴포넌트를 이용한 프로그램 개발이다.

현재 계획으로는 
1. 유튜버의 내용을 그대로 따라간다.
2. 중간 중간 개선이 필요하다고 생각하는 부분은 효율적으로 수정해본다.
3. 실제로 게임의 구실을 하는 프로젝트를 완성한다.
4. 배포 및 서비스 단계까지 실행해본다.
5. 리팩토링을 통해서 전체 소스코드를 최적화한다.

현재 생각을 이러하다. 여유가 있다면 웹에서 실행되도록 진행해보고싶긴한데, 그건 차츰 생각해보자.

우선 프로젝트를 생성한다. 동빈나님은 기본 프로젝트로 생성했지만 추후에 라이브러리를 가져다 쓸 일도 있고, 최종 작업 후 조금 더 편리한 배포를 위해서 메이븐으로 프로젝트를 만들어주자.

프로젝트 생성 후 실행을 위한 메인 메서드부터 생성한다.

나는 다음과 같이 /src/main/java/app/rhythm/player 패키지 하위에 Main 메서드를 위치시켰다.

![main](/assets/images/study/dev/2018/11_main_method.png)

메인 메서드에서 할 일은 프로그램의 실행이다.

~~~java
  public class Main {
    public static final int SCREEN_WIDTH = 1280;
    public static final int SCREEN_HEIGHT = 720;

    public static void main(String[] args) {
        new SenseOfRhythm();
    }
  }

~~~

특별한 내용은 없다.

메인 함수가 할 일은 게임을 실행(new SenseOfRhythm : 게임이름)을 돕는 일 뿐이다.

SenseOfRhythm이라는 클래스의 인스턴스를 생성하면서 게임은 시작된다.

상단에 SCREEN_WIDTH/HEIGHT 값은 Swing 컴포넌트를 통해 그려질 화면의 사이즈다.

기본 사이즈로 1280 * 720 으로 설정했다.

그리고 이제 게임이 시작될 **SenseOfRhythm** 이라는 클래스를 만들어보자.

상단의 스크린샷처럼 우선 Main클래스와 같은 패키지 안에 클래스를 생성했다.

그리고 이 클래스의 내용은 다음을 포함한다.


~~~java
  public class SenseOfRhythm extends JFrame {
    public SenseOfRhythm(){
        setTitle("Sense Of Rhythm");
        setSize(Main.SCREEN_WIDTH, Main.SCREEN_HEIGHT);
        setResizable(false); // 사용자 사이즈 조절 제한
        setLocationRelativeTo(null); // 디스플레이 화면의 정 중앙 위치
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE); // 프로그램 종료 후 실제로 프로세스를 종료시킴
        setVisible(true); // 디스플레이에 노출 (false 설정할 경우 화면이 그려지지 않음)
    }
  }
~~~

현재 클래스에 몇가지 중요한 부분이 있는데,

우선 JFrame을 상속받았다는 것이다. JFrame은 곧 Java가 제공하는 Swing 컴포넌트를 사용하겠다는 것이다.

그리고 

> public SenseOfRhythm() 

생성자를 통해 해당 클래스가 생성되면서 게임에 필요한 기본 세팅을 설정해두었다. 자세한 내용은 소스코드 옆의 주석을 참고하자.

현재 상태그대로 메인 메서드를 실행하면 1280*720 사이즈의 빈 화면이 출력될 것이다.

![display](/assets/images/study/dev/2018/11_basic_display.png)

화면을 띄우는 것 까지 성공했으면 이제 기본 배경 이미지를 삽입해보자.

http://wallpaperswide.com

위 링크를 통해 본인의 입맛에 맞는 이미지를 다운받아보자.(이미지는 현재 화면에 맞는 1280*720 사이즈로 다운받아야 최적화된다.)

 이미지를 띄울때 주의할 점은 기본적인 방법으로 이미지를 띄울때(싱글 버퍼링) 이미지가 바뀌거나, 불러오는 과정에서 버퍼링이 발생하여 깜빡이거나 이미지가 손실되는 문제가 발생할 수있다. <br/>
이러한 문제를 방지하기 위해 [더블 버퍼링]을 사용하는 것이 효율적이다.

>더블 버퍼링이란<br/>
현재 프로그램의 전체 화면 크기에 맞는 이미지를 매 순간마다 생성해서 원하는 컴포넌트만 화면에 출력하는 방식이다. 버퍼에 이미지를 담아서 매 순간마다 이미지를 갱신해주기 때문에, 깜빡이거나 손실되는 이미지를 방지할 수 있어 실시간으로 이미지를 다루는 프로그램에서는 주요하게 사용된다.

우선 다운받은 이미지는 resource경로에 옮겨 두자.

![image_path](/assets/images/study/dev/2018/11_image_path.png)

그리고 이미지파일을 읽어서 화면에 출력시켜주면된다.

SenseOfRhythm 클래스의 멤버변수로 다음과 같이 선언한다.

~~~java
public class SenseOfRhythm extends JFrame {
    private Image screenImage;
    private Graphics screenGraphic;
    private Image introBackground;
}

~~~

그리고 introBackground를 초기화해주면서 이미지를 삽입시켜주자.

~~~java
    public SenseOfRhythm(){
        setTitle("Sense Of Rhythm");
        setSize(Main.SCREEN_WIDTH, Main.SCREEN_HEIGHT);
        setResizable(false); // 사용자 사이즈 조절 제한
        setLocationRelativeTo(null); // 화면의 정 중앙
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE); // 프로그램 종료 후 실제로 프로세스를 종료시킴
        setVisible(true); // 실제로 인터페이스를 표출

        // intro images
        introBackground = new ImageIcon(Main.class.getResource("/images/intro.jpg")).getImage();
    }
~~~

이미지는 게임이 시작되면서(=SenseOfRhythm 인스턴스가 생성되면서) 출력되어야 하기 때문에, 이 클래스가 초기화되는 부분에 넣어주면된다.

주의할점은 resources 하위에 이미지를 저장해놓았기 떄문에 *Main.class.getResource("/images/intro.jpg")* 를 통해서 이미지파일에 접근할 수 있다.

자 이제 거의 다 왔다.

이제 실제로 컴포넌트 화면에 이미지를 그려주어야한다.

이미즤를 가져 왔는데 어디에 어떻게 표현할지를 설정해야하지 않는가?

다음 두가지 메서드를 만든다.

~~~java
    @Override
    public void paint(Graphics g){
        screenImage = createImage(Main.SCREEN_WIDTH, Main.SCREEN_HEIGHT);
        screenGraphic = screenImage.getGraphics();
        screenDraw(screenGraphic);
        g.drawImage(screenImage, 0, 0, null); // 0,0 에 스크린 이미지를 그림
    }


    public void screenDraw(Graphics g){
        // screenDraw 진입시 introBackground를 0,0에 그려준 후 다시 paint함수를 그려준다(this.repaint()) ==> 매 순간마다(프로그램이 종료될때까지) 반
        g.drawImage(introBackground, 0, 0, null); //
        this.repaint(); //
    }

~~~

먼저 paint라는 함수는 JFrame이 생성되면서 가장 먼저 화면에 그려지는 기본 함수이다.

안에 내용을 살펴보면, createImage() 그려질 이미지의 사이즈(x,y)

drawImage는 어떤이미지를, 어떤 위치에 그릴 것인지 설정한다.

그리고 screenDraw 함수를 통해 아까 언급했던 더블 버퍼링을 구현했다.

this.repaint()라는 것은 paint() 함수를 호출하는 것이기 때문에, 

paint() -> screenDraw() -> paint() 로 돌아오는 재귀적 구조로 볼 수 있다.

위와 같이 설정했다면 화면 출력을 위한 구성이 완료되었다.

다시 메인메서드를 실행해보자.

![run](/assets/images/study/dev/2018/11_181101/run.png)

intro 이미지가 잘 나오는 것을 확인할 수 있다.

다음에는 intro 배경음악이 흘러나오는 것을 구현해볼 것이다.

### 출처
- 유튜버 동빈나 : https://www.youtube.com/channel/UChflhu32f5EUHlY7_SetNWw