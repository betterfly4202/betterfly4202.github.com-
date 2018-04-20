---
title: "AssertJ in joda-time 사용법"
layout: post
date: 2018-04-14 19:04
image: 
headerImage: true
tag:
- tdd
- assertj
- joda-time
category: blog
author: betterFLY
description: assertj 에서 joda-time 사용하기
---

![joda](/assets/images/180414/joda.jpg)

## AssertJ assertions for Joda-Time

### 왜 joda-time을 사용해야 하는가
- java.util.Date의 한계
    1. 불변 객체가 아니다( not immutable) <br>
        불행히도 Java의 기본 날짜, 시간 클래스는 불변 객체가 아니다. 앞의 코드에서 Calendar 클래스에 set 메서드를 호출해서 날짜를 지정하고, 다시 같은 객체에 set(int,int) 메서드를 호출해서 수행한 날짜 연산 결과는 같은 인스턴스에 저장되었다. Date 클래스에도 값을 바꿀 수 있는 set 메서드가 존재한다. 이 때문에 Calendar 객체나 Date 객체가 여러 객체에서 공유되면 한 곳에서 바꾼 값이 다른 곳에 영향을 미치는 부작용이 생길 수 있다. 『Effective Java 2nd Edition』(2008)의 저자 Joshua Bloch도 Date 클래스는 불변 객체여야 했다고 지적했다.
        이를 안전하게 구현하려면 이들 객체를 복사해서 반환하는 기법을 권장한다.
       
    2. int 상수 필드의 남용
        <pre> 
            calendar.add(Calendar.SECOND, 2);
        </pre>
        
        첫 번째 파라미터에 Calendar.JUNE과 같이, 전혀 엉뚱한 상수가 들어가도 이를 컴파일 시점에서 확인할 방법이 없다. 이 뿐만 아니라 Calendar 클래스에는 많은 int 상수가 쓰였는데, 이어서 설명할 월, 요일 지정 등에서도 많은 혼란을 유발한다.
    
    3. 헷갈리는 월 지정
        <pre>
            calendar.set(1582, Calendar.OCTOBER , 4);
        </pre>
        
        그런데 월에 해당하는 Calendar.OCTOBER 값은 실제로는 '9'이다. JDK 1.0에서 Date 클래스는 1월을 0으로 표현했고, JDK 1.1부터 포함된 Calendar 클래스도 이러한 관례를 답습했다. 그래서 1582년 10월 4일을 표현하는 코드를 다음과 같이 쓰는 실수를 많은 개발자들이 반복하고 있다.
        
        ~~~java
          @Test
              public void shouldGetDate() {
                  Calendar calendar = Calendar.getInstance();
                  calendar.set(1999, 12, 31);
                  assertThat(calendar.get(Calendar.YEAR)).isEqualTo(2000);
                  assertThat(calendar.get(Calendar.MONTH)).isEqualTo(Calendar.JANUARY);
                  assertThat(calendar.get(Calendar.DAY_OF_MONTH)).isEqualTo(31);
          
                      
              }
          
        ~~~
        
        1999년 12월 31일을 지정하려 했으나, 12월의 상수값은 11이므로 직접 숫자 12를 대입하면 2000년 1월 31일로 넘어간다. 숫자 12 대신 11 혹은 Calendar.DECEMBER 상수로 지정해야 1999년 12월 31일이 된다.
              
        13월을 의미하는 12를 넣어도 Calendar.set() 메서드가 오류를 반환하지 않기 때문에 이런 실수를 인지하기 더욱 어렵다. calendar.setLenient(false)
        메서드를 호출하면 잘못된 월이 지정된 객체에서 IllegalArgumentException을 던져 준다. 그렇게 지정해도 Calendar.set() 메서드가 호출되는 시점이 아니라,
        Calendar.get() 메서드가 호출될 때 Exception이 발생한다는 점도 주의해야한다.
    
    4. 일관성 없는 요일 상수<br/>
        Calendar.get(Calendar.DAY_OF_WEEK) 함수에서 반환한 요일은 int 값으로, 일요일이 1로 표현된다. 따라서 수요일은 4이고, 보통 Calendar.WEDNESDAY 상수와 비교해서 확인한다. 그런데 calendar.getTime() 메서드로 Date 객체를 얻어와서 Date.getDay() 메서드로 요일을 구하면 일요일은 0, 수요일은 3이 된다. 두 개의 클래스 사이에 요일 지정값에 일관성이 없는 것이다.
        Date.getDay() 메서드는 요일을 구하는 메서드로는 이름이 모호하기도 하다. 현재는 사용하지 않는(deprecated) 메서드라서 그나마 다행이다.
        
        ~~~java
          @Test
              @SuppressWarnings("deprecation")
              public void shouldGetDayOfWeek() {
                  Calendar calendar = Calendar.getInstance();
                  calendar.set(2014, Calendar.JANUARY, 1);
          
                  int dayOfWeek = calendar.get(Calendar.DAY_OF_WEEK);
                  assertThat(dayOfWeek).isEqualTo(Calendar.WEDNESDAY);
                  assertThat(dayOfWeek).isEqualTo(4);
                  Date theDate = calendar.getTime();
                  assertThat(theDate.getDay()).isEqualTo(3);
          
                  
              }
        ~~~
        
        Calendar.get(Calendar.DAY_OF_WEEK) 함수에서 반환한 요일은 int 값으로, 일요일이 1로 표현된다. 따라서 수요일은 4이고, 보통 Calendar.WEDNESDAY 상수와 비교해서 확인한다. 그런데 calendar.getTime() 메서드로 Date 객체를 얻어와서 Date.getDay() 메서드로 요일을 구하면 일요일은 0, 수요일은 3이 된다. 두 개의 클래스 사이에 요일 지정값에 일관성이 없는 것이다.
        Date.getDay() 메서드는 요일을 구하는 메서드로는 이름이 모호하기도 하다. 현재는 사용하지 않는(deprecated) 메서드라서 그나마 다행이다.

### Joda-Time 사용


#### maven dependency
~~~xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.9.9</version>
</dependency>

<!-- assertj-test-->
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.9.1</version>
    <scope>test</scope>
</dependency>

~~~

#### gradle
~~~groovy
compile group: 'joda-time', name: 'joda-time', version: '2.9.9' 
testCompile group: 'org.assertj', name: 'assertj-core', version: '3.9.1'
~~~

- **java.utils.Calendar** vs **jodatime**
    - java.utils.Calendar
~~~java
    @Test
    public void 자바_캘린더_기본(){
        Calendar calendar = Calendar.getInstance();
        calendar.set(2000, Calendar.JANUARY, 1, 0, 0, 0);

        SimpleDateFormat sdf =
                new SimpleDateFormat("yyyy-MM-dd E HH:mm:ss.SSS");
        calendar.add(Calendar.DAY_OF_MONTH, 90);
        System.out.println(sdf.format(calendar.getTime()));
    }
~~~

   - jodatime
~~~java
        @Test
        public void 조다타임_기본(){
            DateTime dateTime = new DateTime(2018, 4, 13, 0, 0, 0, 0);
            System.out.println(dateTime.plusDays(90).toString("yyyy-MM-dd E HH:mm:ss.SSS"));
        }
~~~~


#### AssertJ in joda-time

- 데이터타입을 커스터마이징 하여 원하는 조건으로 비교
~~~java
    @Test
    public void 패턴비교(){
        DateTime nowTime = DateTimeFormat.forPattern("yyyy-MM-dd HH:mm:ss").parseDateTime("2018-04-13 00:23:54");
        String result = DateTimeFormat.forPattern("aa").withLocale(new Locale("ko")).print(nowTime);
        assertThat(result).isEqualTo("오전");

        String result2 = DateTimeFormat.forPattern("MM월dd일 HH:mm").withLocale(new Locale("ko")).print(nowTime);
        assertThat(result2).isEqualTo("04월13일 00:23");

    }
~~~
    
- UTC타임과 해당 지역을 비교
~~~java
    @Test
    public void 타임존(){
        DateTime utcTime = new DateTime(2013, 6, 9, 17, 0, DateTimeZone.UTC);
        DateTime cestTime = new DateTime(2013, 6, 10, 2, 0, DateTimeZone.forID("Asia/Seoul"));

        assertThat(utcTime).as("in UTC time").isEqualTo(cestTime);
    }
~~~
        
- 기준시간 이전 혹은 이후 검증 
~~~java
    
    @Test
    public void 시간검증_기본(){
        assertThat(new DateTime("1999-12-30")).isBefore(new DateTime("2000-01-01"));
        assertThat(new DateTime("2000-01-01")).isAfter(new DateTime("1999-12-30"));

        assertThat(new DateTime("2000-01-01")).isBeforeOrEqualTo(new DateTime("2000-01-01"));
        assertThat(new DateTime("2000-01-01")).isAfterOrEqualTo(new DateTime("2000-01-01"));
    }

~~~
    
- 상세 시간을 무시하여 검증
~~~java

    @Test
    public void 시간무시(){
        // ... milliseconds
        DateTime dateTime1 = new DateTime(2000, 1, 1, 0, 0, 1, 0, UTC);
        DateTime dateTime2 = new DateTime(2000, 1, 1, 0, 0, 1, 456, UTC);
        assertThat(dateTime1).isEqualToIgnoringMillis(dateTime2);
        // ... seconds
        dateTime1 = new DateTime(2000, 1, 1, 23, 50, 0, 0, UTC);
        dateTime2 = new DateTime(2000, 1, 1, 23, 50, 10, 456, UTC);
        assertThat(dateTime1).isEqualToIgnoringSeconds(dateTime2);
        // ... minutes
        dateTime1 = new DateTime(2000, 1, 1, 23, 50, 0, 0, UTC);
        dateTime2 = new DateTime(2000, 1, 1, 23, 00, 2, 7, UTC);
        assertThat(dateTime1).isEqualToIgnoringMinutes(dateTime2);
        // ... hours
        dateTime1 = new DateTime(2000, 1, 1, 23, 59, 59, 999, UTC);
        dateTime2 = new DateTime(2000, 1, 1, 00, 00, 00, 000, UTC);
        assertThat(dateTime1).isEqualToIgnoringHours(dateTime2);
    }

~~~

- 기준 시간내에 포함여부 검증 
    <br/> **기준시간.isIn(A, B)** -> 기준시간이 A~B시간 사이에 해당하는지 검증 
~~~java

    @Test
    public void not_in(){
        assertThat(new DateTime("2000-01-01")).isIn(new DateTime("1999-12-22"), new DateTime("2000-01-01")); //A~B까지의 시간안에 포함
        assertThat(new DateTime("2000-01-01")).isNotIn(new DateTime("1999-12-31"), new DateTime("2000-01-02")); //B가 기준 날짜보다 크므로 not in 임
        
        assertThat(new LocalDateTime("2000-01-01")).isIn("1999-12-31", "2000-01-01").isNotIn("1999-12-31", "2000-01-02");
    }

~~~

#### 참고 
- http://d2.naver.com/helloworld/645609
- http://joel-costigliola.github.io/assertj/assertj-joda-time.html
- https://www.lesstif.com/display/JAVA/Joda-Time
- http://jojoldu.tistory.com/26
