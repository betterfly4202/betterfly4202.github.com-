---
title: "Servlet환경에서 DBCP2 사용하기"
categories: "GCP"
tags:
  - DBCP
  - JDBC
---

### 레거시 환경에 DBCP2 적용하기 (not used Spring Framework)

> '레거시 환경' 이라는 표현은 Spring Framework를 사용하지 않고 진행하는 기본 웹 Servlet환경을 의미한다. (doGet / doPost 등을 이용한 API 구성)

레거시환경에서 myBatis를 구현하는 것은 그럭저럭 구현되었다.

사실 개선할점이 많이 보였지만, 말그대로 그럭저럭하게 구현된 것이다.

그런데 소스는 리팩토링하며 수정하면되지만, 기능적으로 문제가 있으면 안될 것이다.

특히나 DB와 관련해선 서비스에 직접적으로 영향을 주는 아주 치명적인 문제를 일으킬 수 있다.

특히 그 중 Connection Pool을 관려하는 환경 구성에 대한 의구심이 끊임없이 들었다.

불안했다.

Connection Pool 인스턴스가 중구난방으로 만들어지는건 아닌지, Pool은 제대로 관리되고 있는지 SqlSession은 낭비되고 있지 않은지...

인터넷에서 제공하는 많은 자료들을 참고해서 필요한 설정은 해두었지만 '이게 맞나..?' 라는 의심이 끊이질 않았다.

그러던 중 내 고민을 해결해 줄 Connection Pool을 관리해주는 Common DBCP2 라는 라이브러리를 찾게되었다.

마이바티스 사용시 거의 기본적으로 사용되는 라이브러리로 거의 필수로 사용되는 라이브러리이다.

이걸 써야겠다.

그런데 문제가 있다.

스프링에서는 <code>SqlSessionFactoryBean</code> 이라는 녀석이 있다.

이 Bean에다가 DB관련 설정을 모두 주입시켜주면, 아주 맘편하게 DB환경을 구축할 수 있다.
DataSource뿐만 아니라, Configuration, Mapper등 DB관련 객체들을 관리해주는 Bean이다.

문제는 DBCP를 이용하기 위해선 myBatis DataSource를 변경해줘야한다.

DataSource환경이 DBCP로 되어야 한다는 것이다.

우리가 일반적으로 사용하는 myBatis-config.xml 정도라고 네이밍하는 myBatis 환경 구성의 모습이다.

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="logImpl" value="LOG4J"/>
        <setting name="cacheEnabled" value="true"/>
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="multipleResultSetsEnabled" value="true"/>
        <setting name="useColumnLabel" value="true"/>
        <setting name="useGeneratedKeys" value="false"/>
        <setting name="autoMappingBehavior" value="PARTIAL"/>
        <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
        <setting name="defaultExecutorType" value="SIMPLE"/>
        <setting name="localCacheScope" value="SESSION"/>
        <setting name="jdbcTypeForNull" value="OTHER"/>
        <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
    </settings>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${mysql_url}"/>
                <property name="username" value="${user}"/>
                <property name="password" value="${password}"/>
                <property name="poolMaximumActiveConnections" value="30"/>
                <property name="poolMaximumIdleConnections" value="30"/>
                <property name="poolTimeToWait" value="20000"/>
                <property name="poolPingEnabled" value="true"/>
                <property name="poolPingQuery" value="select 1"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="mybatis/mappers/com/sample-mapper.xml"/>
    </mappers>
</configuration>
~~~

이 환경에서 <code><environment><code>의 DBCP를 적용하기 위해선 DataSource만 변경해주면된다.

역시 문제는 위 상황에서 dataSource에 DBCP를 넣는다고 인식되지 않는다.

우선 메이븐을 통해 dbcp2를 가져오자.

~~~xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-dbcp2</artifactId>
    <version>2.5.0</version>
</dependency>
~~~

깔끔하게 xml을 통해서 구현하고 싶은데, 방법을 모르겠다.

그래서 자바에서 구현했다.

~~~java
    private static final int INITIALIZE_POOL_SIZE = 10;
    private static final int MAX_CONNECTION_POOL_SIZE = 10;
    private static final int MAX_IDLE_POOL_SIZE = 10;
    private static final long MAX_CONNECTION_WAIT_MS = 30000L;

    public static BasicDataSource dbcpDataSource() {
        BasicDataSource dbcp = new BasicDataSource();
        dbcp.setDriverClassName("com.mysql.jdbc.Driver");
        if(getProperties("service").equals("local")){
            dbcp.setUrl(getProperties("baseUrl"));
            dbcp.setUsername(getProperties("userName"));
            dbcp.setPassword(getProperties("userPassword"));
        }else{
            dbcp.setUrl(getJdbcConnectUrl());
        }

        dbcp.setInitialSize(INITIALIZE_POOL_SIZE);
        dbcp.setMaxTotal(MAX_CONNECTION_POOL_SIZE);
        dbcp.setMaxIdle(MAX_IDLE_POOL_SIZE);
        dbcp.setMaxWaitMillis(MAX_CONNECTION_WAIT_MS);
        dbcp.setTestWhileIdle(true);
        dbcp.setValidationQuery("SELECT 1");

        return dbcp;
    }
~~~

> source에서 사용하는 getProperties() 함수는 메이븐 프로파일별로 설정된 각각의 프로파일들을 가져오기 위한 함수이다.

사실 이렇게하고 JDBC접속을 최종적으로 도와주는 SqlSessionFactory 클래스만 다루면 되는데, 문제는 myBatis-config.xml 에 있는 mapper들을 가져와야한다.

(사실 mapper로 쿼리 쓰자고 myBatis쓰는건데 mapper를 못가져오면 도루묵이다.)

중요한 SqlSessionFactory부분인데, 

~~~java
    public static synchronized SqlSessionFactory getInstance(){
        if(sqlSessionFactory == null){
            DataSource dataSource = MyBatisConfig.dbcpDataSource();
            TransactionFactory trxFactory = new JdbcTransactionFactory();
            Environment env = new Environment("development", trxFactory, dataSource);
            Configuration config = new Configuration(env);

            sqlSessionFactory = new SqlSessionFactoryBuilder().build(config);
        }
        sqlSessionFactory.getConfiguration();
        return sqlSessionFactory;
    }
~~~

벌써 뭔가 허전하다.

sqlSessionFactory에 'config' 를 넣어줬는데 Configuration에 실제로 유효한 값은 DataSource 밖에 없다.
나머지 TransactionFactory, Environment 등의 값은 어디에도 없다.

난 DataSource만 DBCP를 적용하고 나머지는 myBatis-config.xml 에 설정한 값을 사용하고 싶다.

그렇게 디버깅을 하며 삽질하던 중 방법을 찾을 수 있었다.

<code>SqlSessionFactoryBuilder</code> 결국 이 클래스에 configuration을 전달해줘야 한다는 건데, 얘가 어떻게 세팅되어야 하는지 클래스 속을 들여다 보았다.

~~~java
      public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
        SqlSessionFactory var5;
        try {
            XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
            var5 = this.build(parser.parse());
        } catch (Exception var14) {
            throw ExceptionFactory.wrapException("Error building SqlSession.", var14);
        } finally {
            ErrorContext.instance().reset();

            try {
                inputStream.close();
            } catch (IOException var13) {
                ;
            }

        }

        return var5;
    }
~~~

실제로 SqlSessionFactoryBuilder 클래스의 build 메서드의 로직이다.

생각보다 단순했다. xml파일 자체를 InputStream으로 읽어들여 XML Parsing을 하는 것이다.

똑같이 해주면된다.

그래서 했다.

### Configuration

~~~java
  private static Configuration dbcpConfiguration(){
    DataSource dataSource = MyBatisConfig.dbcpDataSource();
    TransactionFactory trxFactory = new JdbcTransactionFactory();
    Environment env = new Environment("dbcp",trxFactory, dataSource);

    InputStream inputStream = MyBatisConfig.class.getClassLoader().getResourceAsStream("mybatis/mybatis-config.xml");
    XMLConfigBuilder parser = new XMLConfigBuilder(inputStream);
    Configuration config = parser.parse();
    config.setEnvironment(env);

    return config;
  }
~~~

### 최종 SqlSessionFactory 인스턴스 생성 모습

~~~java
  public static synchronized SqlSessionFactory getInstance(){
      if(sqlSessionFactory == null){
          sqlSessionFactory = new SqlSessionFactoryBuilder().build(dbcpConfiguration());
      }
      sqlSessionFactory.getConfiguration();
      return sqlSessionFactory;
  }
~~~

정리하자면, 내가 필요했던건 myBatis에서 기본으로 사용하는 DataSource접근이 아닌, 내가 원하는 DataSource로 세팅하여 접근하는 것이었다.
mybatis의 configuration xml내에서 직접 접근할 수 있는 방법이 떠오르질 않아,
자바에서 직접 접근이 가능하도록 삽질을 했다.

꽤 오랜 삽질끝에 구현은 되었는데 좋은 방법인지는 또 다른 시행착오를 겪어봐야할 것 같다.