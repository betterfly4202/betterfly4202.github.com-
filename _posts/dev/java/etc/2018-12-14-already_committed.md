---
title: "Response has already been committed"
categories: "Java"
tags:
  - 
---

보안 이슈 작업을 하며, 전체 요청에 대한 필터링이 필요했다.

처음엔 로그인 시도시에만 IP 유효성 검사를 처리했는데, 전체 페이지에 접근시 처리될 수 있도록 구현이 필요했다.

방법을 고민하던 중 필터링을 통해서 모든 url접근 (/*)에서 IP검증 로직을 태우면 될 것 같다.

그리고 간단히 로직을 구현 후 서버를 돌렸는데 처음 보는 낯선 예외상황이 발생했다.

> java.lang.IllegalStateException: Response has already been committed

Response가 이미 committed 되었다고 한다.

무슨 소린진 알겠는데 왜 그런지 모르겠다. 무엇이 문제일까.

결과적으로 필터하는 과정에서 sendRedirect 시키는 필터링의 문제인 것을 찾을 수 있었다.

그리고 다음과 같이 web.xml 에서 적용 시켰는데 서버가 구동되면서 해당 필터의 sendRedirect를 3번 호출하면서 문제가 발생됐다는 것을 확인했다.

다시 말해 Response를 이미 받아 온 후 해당 Response에 중복된 응답을 요청하는 경우 발생한다는 것이다.

~~~
    <filter>
        <filter-name>AccessIP</filter-name>
        <filter-class>com.wemakeprice.push.common.AccessIpFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>AccessIP</filter-name>
        <url-pattern>/login.jsp</url-pattern>
    </filter-mapping>
~~~

왜..

왜 3번 호출됐을까.

이 문제를 해결하기 위해선 2가지 해결방안이 있는 것 같다.

- 3번 호출되는 원인을 찾아 1번만 호출하도록 변경한다.
- 호출할때 예외처리를 한다.

난 근본적인 원인을 찾아서 1번만 호출되도록 하게 하고 싶었다.

if문을 겹겹이 하여 어설픈 예외처리로 임시방편으로 막는듯한 느낌은 왠지 찝찝하다.

그런데 이 문제는... 서버가 구동되면서 반복해서 index 페이지를 3회 호출하는 것인데, 도저히 파악이 힘들었다.

특히 개발환경이 GCP내의 App Engine 환경이기 떄문에 was서버 설정을 확인할 수도 없는 노릇이다.

---

결국 조금 다른 방법으로 우회하여 처리했다.

처음 웹에 진입시 기본 <code>index.jsp</code> 호출시 

~~~javascript
    location.href="/login.jsp"
~~~

와 같이 login화면으로 보내버린다.

그리고 이동된 로그인화면에서 java코드를 구현해 놓았다.

~~~jsp
<%
    if(!ApiUtils.checkIPValidation(request)){
        response.sendRedirect("/error/access.jsp");
        return;
    }
%>
~~~

로그인 화면에서 ip유효성을 검사하여 에러페이지로 보내버리면 간단히 처리되는 것이다.

고생스럽게 고민한 것 치곤, 쉬운 방법으로 정리가 된 것 같아 허무했지만. 반대로 쉽게 처리할 수 있는 것을 너무 복잡하고 어렵게 접근하는 습관이 안좋다는 것을 다시 한 번 느낄 수 있었다.