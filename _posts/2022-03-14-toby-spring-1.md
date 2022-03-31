---
title:  "[SPRING] 토비 스프링 vol1 1장"
excerpt: "토비의 스프링"

categories: Java
tags:
  - [SPRING]

toc: true
toc_sticky: true
 
date: 2022-03-14
last_modified_at: 2022-03-14
author_profile: false     
---

## 초난감 DAO 관심사항의 분리

 - 미래를 준비하는데 있어 가장 중요한 것은 변화에 어떻게 대비할 것인가이다. 
 가장 좋은 대책은 변화의 폭을 최소한으로 줄여주는 것이다. 
 - 변화가 한가지 관심에 집중돼서 일어난다면, 우리가 해야 할 것은 한가지 관심이
 한 군데에 집중되게 하는 것이다. 

 ## USerDao 의 관심사항

  - 첫째는 DB와 연결을 위한 커넥션(어떤 DB를 쓰고, 어떤 드라이버를 사용하고, 계정이 뭐고,,)
  - 둘째는 SQL 문장을 담을 Statement를 만들고 실행하는것
  - 셋째는 작업한 리소스를 close 하는 것

```java
public void add(User user) throws ClassNotFoundException, SQLException {
        Class.forName("org.h2.Driver");
        Connection c = DriverManager.getConnection("jdbc:h2:tcp://localhost/~/test", "sa",
                "");

        PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }

public User get(String id) throws ClassNotFoundException, SQLException {
        Class.forName("org.h2.Driver");
        Connection c = DriverManager.getConnection("jdbc:h2:tcp://localhost/~/test", "sa",
                "");
        PreparedStatement ps = c
                .prepareStatement("select * from users where id = ?");
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();
        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        c.close();

        return user;
    }

```

 - 리팩토링 기법중에 메소드 추출을 이용하여 connection 가져오는 부분을 추출
 이로써, connection 가져오는 로직의 중복을 제거하였다. 

```java
public void add(User user) throws ClassNotFoundException, SQLException {
        Class.forName("org.h2.Driver");
        Connection c = getConnection();

        PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }

    private Connection getConnection() throws SQLException {
        return DriverManager.getConnection("jdbc:h2:tcp://localhost/~/test", "sa",
                "");
    }
```

 - 만약 사이트 별로 DB커넥션 정보를 다르게 설정해주어야 한다면?
 - 기존에 같은 클래스에 다른 메소드로 분리됐던 DB커넥션 연결이라는 관심을
 이번에는 상속을 통해 서브클래스로 분리한다. 

![image1](/assets/images/page8/image1.PNG)

 - N사와 D사가 각각 다른 커넥션 로직이 구현되어야 한다면, UserDao 를 상속받아, 
 커넥션을 각 사에 맞게 구현만 해주면 된다. add, get 기능은 공통으로 사용하되, 커넥션 가져오기만
 다르게 구현.
 - 이렇게 슈퍼 클래스에 기본적인 로직의 흐름(커넥션가져오기,  SQL 생성, 실행, 반환)을 만들고, 
 그 기능의 일부를 추상 메소드나 오버라이딩이 가능한 protected 메소드 등으로 만든뒤
 서브 클래스에서 이런 메소드를 필요에 맞게 구현해서 사용하도록 하는 방법을 템플릿 메소드 패턴이라고 한다.
 - 템플릿 메소드 패턴은 상속을 통해 슈퍼클래스의 기능을 확장할 때 사용되는 가장 대표적인 방법이다. 
 변하지 않는 기능은 슈퍼클래스에 만들고, 자주 변경되거나 확장될 기능은 서브클래스에서 만들도록 한다. 

```java
public abstract class UserDao {

public abstract Connection getConnection() throws SQLException ;

}

static class NUserDao extends UserDao{

        @Override
        public Connection getConnection() throws SQLException {
            //N사 커넥션 로직 구현
            return null;
        }
    }

    static class DUserDao extends UserDao{

        @Override
        public Connection getConnection() throws SQLException {
            //D사 커넥션 로직 구현
            return null;
        }
    }

```

 - 이번에는 서브클래스가 아니라 아예 별도의 클래스로 분리해보자.

 ![image1](/assets/images/page8/image2.PNG)

```java
public class SimpleConnectionMaker {

    public Connection getConnection() throws SQLException {
        return DriverManager.getConnection("jdbc:h2:tcp://localhost/~/test", "sa",
                "");
    }
}

public class UserDao {

    private SimpleConnectionMaker simpleConnectionMaker;

    public UserDao(){
        simpleConnectionMaker = new SimpleConnectionMaker();
    }

    add();

    get();

}    
```



 - 여기서 문제는 DB커넥션을 제공하는 구체클래스 정보를 userDao 가 알고 있어야 한다는 것이다.
 만약 D사에서 SimpleConnectinMaker 를 사용할 수 없다면, 결국 userDao 가 수정되어야 한다. 
 - 구체클래스 대신 인터페이스를 사용하자.

![image1](/assets/images/page8/image3.PNG)

 - 이제 UserDao 는 자신이 사용할 클래스가 어떤 것인지 몰라도 된다. 단지 인터페이스를 통해
 원하는 기능을 사용하기만 하면 된다. 

 ```java
 public interface ConnectionMaker {
    public Connection getConnection() throws SQLException ;
}

public class UserDao {
    private ConnectionMaker connectionMaker;

    public UserDao(){
        connectionMaker = new DConnectionMaker();
    }

    static class DConnectionMaker implements ConnectionMaker{
        @Override
        public Connection getConnection() throws SQLException {
            return null;
        }
    }
    
 ```

  - 인터페이스를 사용하였지만, 아직도 userDao 가 구체클래스를 알고 있어야 한다..ㅜ
  - userDao가 직접 ConnectionMaker 객체를 생성하지 않고, 외부에서 주입받는 형태로 바꾸자. 
  - userDao는 ConnectionMaker 생성에 대한 책임을 클라이언트로 넘겼다. 

```java
public class UserDao {
    private ConnectionMaker connectionMaker;

    public UserDao(ConnectionMaker connectionMaker){
        connectionMaker = connectionMaker;
    }
```

 - 개방폐쇄원칙(OCP) : 확장에는 열려 있고, 변경에는 닫혀있다.
 - userDao 는  전혀 영향없이, DB연결 방법이라는 기능을 확장할 수 있게 되었다. 
 - 낮은 결합도 : 결합도란 하나의 오브젝트가 변경이 일어날때 관계를 맺고 있는
 다른 오브젝트에 변화를 요구하느 정도, 낮은 결합도란 하나의 변경할때
 마치 파문이 이는 것처럼 여타 모듈과 객체로 변경에 대한 요구가 전파되지 않는 상태

 ## 전략패턴

  - 위 구조는 디자인 패턴의 꽃이라고 불리우는 전략패턴에 해당한다고 볼 수 있다. 
  - 필요에 따라 변경이 필요한 알고리즘을 인터페이스를 통해 통째로 외부로 분리

## 오브젝트 팩토리

  - 팩토리의 역할 : 클라이언트가 UserDao 와 ConnectionMaker객체를 만들고 두 오브젝트를
  연결해서 사용할 수 있게 관계가 맺어주는 것을 하고 있었는데, 이 기능을 담당할 클래스를 별도로
  만들어보자. 이 클래스의 역할은 객체의 생성 방법을 결정하고 그렇게 만들어진 오브젝트를 돌려주는
  것인데, 이런 오브젝트를 흔히 팩토리라고 부른다.

```java
public class DaoFactory {
    public UserDao userDao(){
        ConnectionMaker connectionMaker = new UserDao.DConnectionMaker();
        UserDao userDao = new UserDao(connectionMaker);
        return userDao;
    }
}
```

  - 이렇게 분리된 오브젝트들의 역할과 관계를 분석해보자. UserDao 와 ConnectionMaker는 각각
  애플리케이션의 핵심적인 데이터 로직과 기술 로직을 담당하고 있고, DaoFactory는 
  이런 애플리케이션의 오브젝트들을 구성하고 그 관계를 정의하는 책임을 맡고 있음을 알 수 있다.

  - DaoFactory를 분리하여 얻을 수 있는 장점은 다양하다. 그 중에서도 애플리케이션의 컴포넌트
  역할을 하는 오브젝트와 애플리케이션의 구조를 결정하는 오브젝트를 분리했다는데 가장 의미가 있다. 

   - DaoFactory 진화

```java
   public class DaoFactory {
    public UserDao userDao(){
        return new UserDao(connectionMaker());
    }
    public AccountDao accountDao(){
        return new AccountDao(connectionMaker());
    }
    public MessageDao messageDao(){
        return new MessageDao(connectionMaker());
    }

    // 객체생성 중복을 제거하기 위해 분리
    public ConnectionMaker connectionMaker(){
      return new DConnectionMaker();
    }
}
```

## 제어관계 역전

 - 라이브러리는 애플리케이션의 코드를 직접 제어한다. 반면 프레임워크는 거꾸로
 애플리케이션 코드가 프레임워크에 의해 사용된다. 

## 오브젝트 팩토리를 이용한 스프링 IoC

  - 스프링에서는 빈의 생성과 관계설정 같은 제어를 담당하는 IoC 오브젝트를 
  빈 팩토리라고 부른다. 주로 애플리케이션 컨텍스트를 사용한다.(애플리케이션 전반에
   걸쳐 모든 구성 요소의 제어 작업을 담당하는 IoC 엔진이라는 의미)
   
```java

@Configuration
public class DaoFactory {

    @Bean
    public UserDao userDao(){
        return new UserDao(connectionMaker());
    }

    @Bean
    public ConnectionMaker connectionMaker(){
        return new UserDao.DConnectionMaker();
    }
}

class UserDaoTest{
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
        UserDao dao = context.getBean("userDao", UserDao.class);
        System.out.println(dao);
    }
}

```

 - 애플리케이션 컨텍스트가 동작하는 방식

![image1](/assets/images/page8/image4.PNG)

 - DaoFactory를 사용했을때와 비교해서 얻을 수 있는 장점
 - 클라이언트는 구체적인 팩토리 클래스를 알 필요가 없다. 
 - 종합 IoC 서비스 제공(생성시점, 전략, 후처리, 인터셉팅등등)
 - 빈을 검색하는 다양한 방법 제공

## 싱글톤

 - 애플리케이션 컨텍스트는 싱글톤을 저장하고 관리하는 싱글톤 레지스트리이다. 
 - 왜 싱글톤? 이는 스프링이 주로 적용되는 대상이 자바 엔터프라이즈 기술을 사용하는 서버 환경이기 때문이다. 매번 클라이언트에서 요청이 올때마다 객체가 생성된다면 너무 비효율적일것.

## 싱글톤의 한계

  - private 생성자를 갖고 있기 때문에 상속이 어렵다. 
  - 테스트 하기 어렵다. 
  - 하나만 만들어지는 것을 보장하지 못한다. 
  - 스프링은 이런 단점들 때문에 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공한다. 

## 의존관계 주입

 - 인터페이스를 통한 의존관계를 갖고 있어야 한다. 
 - 인터페이스를 사용하면 런타임 시점에 의존관계가 결정된다. 
 - 인터페이스를 사용함으로써, 부가기능도 쉽게 추가할수 있다. 

```java
public class CoutingConnectionMaker implements ConnectionMaker{

  int count = 0;
  private ConnectionMaker realConnectionMaker;

  public CountingConnectionMaker(ConnectionMaker realConnectionMaker){
    this.realConnectionMaker = realConnectionMaker;
  }

  public Connection makeConnection(){
    this.count++;
    return realConnectionMaker.makeConnection();
  }

  public int getCounter(){
    return this.count;
  }

}
``` 
## XML을 이용한 설정

![image1](/assets/images/page8/image5.PNG)


## DataSource

 - ConnectionMaker 는 DB커넥션을 생성해주는 기능 하나만을 정의
 - 자바에서는 DB커넥션을 가져오는 오브젝트의 기능을 추상화해서 비슷한 용도로 사요앟ㄹ 수 있게 만들어진 DataSource 라는 인터페이스가 존재함
 - 아래 자바 설정을 이용하여 UserDao 에 DataSource 인터페이스를 적용하고, 
 SimpleDriverDataSource의 오브젝트를 DI로 주입해서 사용할 준비가 끝났다.

```java
@Configuration
public class DaoFactory {

    @Bean
    public DataSource dataSource(){
        SimpleDriverDataSource dataSource = new SimpleDriverDataSource();
        dataSource.setDriverClass(org.h2.Driver.class);
        dataSource.setUrl("jdbc:h2:tcp://localhost/~/test");
        dataSource.setUsername("sa");
        dataSource.setPassword("");

        return  dataSource;
    }

    @Bean
    public UserDao userDao(){
        UserDao userDao = new UserDao();
        userDao.setDataSource(dataSource());
        return userDao;
    }
}
```

 - 아래는 xml 설정 방식이다. 

 ```xml
 <?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
						http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
        <property name="driverClass" value="org.h2.Driver" />
        <property name="url" value="jdbc:h2:tcp://localhost/~/test" />
        <property name="username" value="sa" />
        <property name="password" value="" />
    </bean>

    <bean id="userDao" class="com.sk.user.daoV1.UserDao">
        <property name="dataSource" ref="dataSource" />
    </bean>
</beans>
 ```



# Reference

> - 토비의 스프링 3.1 vol1 스프링의 이해와 원리