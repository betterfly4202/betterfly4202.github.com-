---
title: "SAX Parser Exception"
categories: "JAVA"
tags:
  - XML
  - JDBC
---

driver : com.mysql.jdbc.GoogleDriver

Cannot create JDBC driver of class 'com.mysql.jdbc.GoogleDriver' for connect URL 'jdbc:mysql://google/scheme?useSSL=false&amp;cloudSqlInstance=database&amp;socketFactory=com.google.cloud.sql.mysql.SocketFactory&amp;autoReconnect=true&amp;characterEncoding=UTF-8'

java.sql.SQLException: No suitable driver


이런 문제


driver : com.mysql.jdbc.Driver

### Cause: java.sql.SQLException: Cannot create PoolableConnectionFactory (Communications link failure
The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server.)

Caused by: java.net.UnknownHostException: google


[SAXParser]

Simple API for XML

이름만봐도 XML을 위한거다.

그렇다 XML과 같은 Mark-up Language에서는 특수문자를 파싱하기 위해선 인식하기 위해선 다른 방법으로 치환하여 표현한다.

특히 많이 사용하는 방법은 아래와 같다.
----------------------------
특수문자   |  바꿔주어야할 문자 
----------------------------
"        |    &quot;
----------------------------
&        |    &amp
----------------------------
'        |    &apos
----------------------------
<        |    &lt;
----------------------------
>        |    &gt;
----------------------------




