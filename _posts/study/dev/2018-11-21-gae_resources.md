---
title: "[Google App Engine] include HTML in JSP"
categories: "GCP"
tags:
  - Google Cloud Platform
  - Google App Engine
  - JSP
---

황당한 일을 겪었다.

진행 중인 웹서비스에서 Google App Engine(이하 GAE)에 배포 후 팝업화면 전체가 노출되지 않는 것이다.

로컬에서 테스트시에는 문제가 없던 것들이, GAE 배포만 하면 모든 팝업창이 나타나지 않았다.

어떤 에러로그도 발생하지 않고 아무런 반응이 없다.
네트워크를 보면, 아무리 팝업창을 클릭해도 API가 호출되지 않는다.

API통신 자체가 되지 않는 다는 것은 팝업창에 문제가 있으리라.

꽤 오랜 삽질과 인고의 시간 결과, 현재 우리 프로젝트의 Front-end부분의 팝업창 구현은 모두 각각의 HTML을 include시켜 이벤트가 발생하도록 구현되어 있다.

예를들어 '히스토리' 라는 메뉴가 있다면, history/index.jsp 라는 각각의 메인 서비스에 대한 jsp 기본 화면을 그려놓고.

그 안에 

~~~html
<jsp:include page="modal-item-detail.html" flush="true"></jsp:include>
~~~

이와 같은 방식으로 각각의 HTML로 구현해놓은 HTML파일들을 <code> jsp:include </code>를 통해서 삽입했다.

문제는 우리 프로젝트르를 패키징한 후 이렇게 include한 부분이 모두 제거되어 있었던 것이다!!

### 정상적으로 include된 소스
![include_html](/assets/images/study/dev/2018/11_included_modal.png)

### include되지 않은 소스
![not_include_html](/assets/images/study/dev/2018/11_not_included_modal.png)

이처럼 include 자체에 문제가 있는 것으로 보였다.

[jsp:include와 <%@ include%> 의 차이>](http://yongblog.tistory.com/entry/jspinclude-%EC%99%80-include-%EC%B0%A8%EC%9D%B4)

위 링크를 보면, 'jsp:include 특징 중 서블릿 컨테이너에 따라 HTML 페이지는 include가 안될 수 있다.' 고 한다. 사실 기존에 되던 방식이기 때문에(서블릿 컨테이너가 바뀐것이 없기 때문에) 이걸로 될까 싶어 <code><%@ include ></code> 방식으로 진행해봤지만 결과는 똑같았다.

...

그러다 문득 servlet 이라는 힌트까 떠올랐다.

app engine에 올릴때 설정을하는 <code>appengine-web-app</code> 의 구성이 의심되었다.

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
    <version>v1-snapshot</version>
    <runtime>java8</runtime>
    <threadsafe>true</threadsafe>
    <use-google-connector-j>true</use-google-connector-j>

    <warmup-requests-enabled>false</warmup-requests-enabled>

    <env-variables>
        <env-var name="DEFAULT_ENCODING" value="UTF-8"/>
    </env-variables>

    
    <resource-files>
        <include path="/**.xml"/>
    </resource-files>
    

    <system-properties>
        <property name="cloudsql" value="${mysql_url}"/>
    </system-properties>

    <static-files>
        <include path="/**.html"></include>
        <include path="/**.json"></include>
        <include path="/**.js"></include>
        <include path="/**.css"/>
        <include path="/**.scss"/>
        <include path="/**.png"/>
        <include path="/**.jpg"/>
        <include path="/**.ico"/>
        <include path="/**.gif"/>
        <include path="/**.woff"/>
        <include path="/**.woff2"/>
        <include path="/**.svg"/>
        <include path="/**.eot"/>
        <include path="/**.ttf"/>
    </static-files>

   ...
</appengine-web-app>
~~~

대략적으로 이와 같이 구현되어있었는데...

~~~xml
<resource-files>
    <include path="/**.xml"/>
</resource-files>
~~~

이 부분은 내가 myBatis를 도입하면서 resources/ 하위에 mapper를 사용하기 위해 설정해놓은 것이다.

사실 jsp/html 쪽에 관련된 작업을 한 것이 아무것도 없는데, 그쪽에서 문제가 난다는 것은 배포 설정에 문제가 있다는 것밖에 안된다.

지워봤다.

그리고 다시 <code>mvn clean package</code>

일단 메이븐 패키징은 문제없이 된다.

혹시 myBatis가 mapper를 찾지 못할까 걱정되었다. 하지만 전혀 문제없이 쿼릴르 날리고 있다.

허무했지만 침착하게 dev deploy를 진행했다.

<code>-Pdev clean appengine:deploy</code>

역시 정상적으로 실행되었고. 팝업창도 정상적으로 출력된다.

당연히 html도 문제없이 include되었다..

~~~xml
<resource-files>
    <include path="/**.xml"/>
</resource-files>
~~~

resource파일 중 xml만 사용하고 나머지는 제외시키는 것일까?

이 설정이 jsp:include 자체를 막아버린 것일까?

문제는 해결했지만, 근본적인 원인은 파악되지 않아 여전히 찜찜한 상황이다.

우선 기록해두고, 계속 트래킹해보리라.