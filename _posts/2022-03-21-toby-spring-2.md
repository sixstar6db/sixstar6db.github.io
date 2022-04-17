---
title:  "[spring] 토비 스프링 vol1 2장"
excerpt: "토비의 스프링"

categories: Java
tags:
  - [spring]

toc: true
toc_sticky: true
 
date: 2022-03-21
last_modified_at: 2022-03-21
author_profile: false     
---

## 테스트

 - 테스트 기술은 만들어진 코드를 확신할 수 있게 해주고, 변화에 유연하게 대처할 수 있는 자신감을 심어준다.
 - 테스트는 작은 단위로 하는 것이 좋다.(단위 테스트)
 - 단위테스트를 하는 이유는 개발자가 설계하고 만든 코드가 원래 의도대로 동작하는지 개발자 스스로 빨리 확인받기 위해서다.
- 코드의 기능을 모두 점검할 수 있는 포괄적인 테스트를 만들어두면, 
애플리케이션에 어떤 과감한 수정을 하고 나서도 테스트를 모두 돌려보고 나면 안심이 된다.

- 아래는 SpringBoot 2.3 , Junit5를 사용한 테스트 코드이다. 
- SpringBoot2.1 부터는 Spring test 에 junit5가 포함되어 있다고 한다.
- @ExtendWith(SpringExtension.class)를 선언하면 Spring Test Context Framework 를 Junit5 프로그래밍에 포함시킬수 있다고 함.
- 스프링의 junit  확장 기능은 테스트가 실행되기 전에 딱 한번만 애플리케이션 컨택스트를 만들어두고, 테스트 오브젝트가 만들어질때마다 특별한 방법을 이용해 애플리케이션 컨텍스트 자신을 테스트 오브젝트의 특정 필드에 주입한다. 
- applicationContext.xml 파일은 src/test/resources 에 있는게 우선순위가 더 높다.
테스트 전용 설정파일을 만들어 테스트 하면 좋다. dao경우, test설정파일에는 테스트용 DB정보로 설정. 
- 테스트는 코드에 변경사항이 없다면 항상 동일한 결과를 내야 한다. dao.deleteAll();
을 통해서 초기상태를 동일하도록 한다. deleteAll 이 싫다면, insert나 update 테스트 완료 후에
원래 상태로 돌리는 쿼리를 수행해도 괜찮다. 

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(locations="/config/applicationContext.xml")
//@ContextConfiguration(classes={DaoFactory.class})
class UserDaoTest {

    @Autowired
    private UserDao dao;

    @Test
    public void addAndGet() throws SQLException {
        //dao = this.context.getBean("userDao", UserDao.class);
        User user1 = new User("1", "sangkook", "1111");
        User user2 = new User("2", "miyu", "1222");

        dao.deleteAll();

        assertThat(dao.getCount()).isEqualTo(0);

        dao.add(user1);
        assertThat(dao.getCount()).isEqualTo(1);

        dao.add(user2);
        assertThat(dao.getCount()).isEqualTo(2);
        User user = dao.get("2");
        assertThat(user.getId()).isEqualTo(user2.getId());
    }

    @Test
    public void getUserFailure() throws SQLException {

        dao.deleteAll();
        EmptyResultDataAccessException e = assertThrows(EmptyResultDataAccessException.class,
                ()->dao.get("1"));
        assertThat(e.getMessage()).isEqualTo("empty data");
    }
}
```

 - 꼭 스프링 컨텍스트가 필요한 상황이 아니라면, 스프링 없이 테스트 하는게 제일 좋다. 
 - 테스트는 항상 빠르게 할 수 있는 것이 좋음.

```java
class UserDaoTestWithoutSpring {

    private UserDao dao;

    @Test
    public void addAndGet() throws SQLException {

        dao = new UserDao();

        DataSource dataSource = new SingleConnectionDataSource(
                "jdbc:h2:tcp://localhost/~/test", "sa","", true
        );
        dao.setDataSource(dataSource);
        User user1 = new User("1", "sangkook", "1111");
        User user2 = new User("2", "miyu", "1222");

        dao.deleteAll();

        assertThat(dao.getCount()).isEqualTo(0);

        dao.add(user1);
        assertThat(dao.getCount()).isEqualTo(1);

        dao.add(user2);
        assertThat(dao.getCount()).isEqualTo(2);
        User user = dao.get("2");
        assertThat(user.getId()).isEqualTo(user2.getId());
    }
}
``` 

## 테스트 주도 개발

 - 실패한 테스트를 성공시키기 위한 목적이 아닌 코드는 만들지 않는다. 
 - 테스트 작성과 이를 성공시키는 코드를 만드는 작업의 주기를 가능한한 짧게 가져가도록 권장한다. 

 ## Junit5 어노테이션

  - @BeforeAll : 모든 테스트 실행 전 최초 한번 실행
  - @BeforeEach : 테스트 실행할 때마다 테스트 전에 실행
  - @AfterEach : 테스트 종료할때마다 테스트 이후 실행
  - @AfterAll : 모든 테스트 종료 후 마지막 실행

## 정리
 
- 테스트는 자동화 되고, 빠르게 실행될수 있어야 한다. 
- 테스트 결과는 일관성이 있어야 한다. 
- 테스트 코드도 적절히 리팩토링이 필요하다. 
- 스프링 테스트 컨텍스트 프레임워크를 이용하면 테스트 성능을 향상시킬수 있다.
- 동일한 설정파일을 사용하는 테스트는 하나의 애플리케이션 컨텍스트를 공유한다. 
- 기술의 사용방법을 익히고 이해를 돕기 위해 학습 테스트를 작성하자. 
- 오류가 발견될 경우 그에 대한 버그 테스트를 만들어두면 유용하다. 

# Reference

> - 토비의 스프링 3.1 vol1 스프링의 이해와 원리
