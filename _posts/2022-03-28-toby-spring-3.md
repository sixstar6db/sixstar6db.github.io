---
title:  "[spring] 토비 스프링 vol1 3장"
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

## DAO 예외처리 코드

 - 오리지널 db 접속 코드에서는 try/catch 문으로 장황하게 코드가 나열되어 있다. 

```java
public void deleteAll() throws SQLException {

        Connection c = null;
        PreparedStatement ps = null;

        try{

            c = connectionMaker.getConnection();

            ps = c.prepareStatement("delete from users");
            
            ps.executeUpdate();
            
        }catch(SQLException e){
            throw e;
        }finally{
            if(ps!=null) ps.close();
            if(c!=null) c.close();
        }
    }
```


## 분리와 재사용을 위한 디자인 패턴 적용

  - 1장에서는 메소드 추출 - 템플릿메소드 패턴 - 전략패턴 방식으로 차례차례 소스를 개선해 나갔다. 
  - 위 소스에서 보면, `ps = c.prepareStatement("delete from users")` 이 부분은 변하는 부분이고(쿼리가 변할수 있으니) 나머지는 변하지 않는 부분이다. 
  - 먼저 메소드 추출을 해보자

```java
public void deleteAll() throws SQLException {
        Connection c = null;
        PreparedStatement ps = null;

        try{
            c = connectionMaker.getConnection();
            ps = makeStatement(c);
            ps.executeUpdate();

        }catch(SQLException e){
            throw e;
        }finally{
            if(ps!=null) ps.close();
            if(c!=null) c.close();
        }
    }

    private PreparedStatement makeStatement(Connection c) throws SQLException {
        return c.prepareStatement("delete from users");
    }
```

 - 메소드 추출 된 부분은 다른 곳에서 재사용이 가능해야 하는데, 위 소스는 그렇지 못하다.

 - 템플릿 메소드 패턴을 적용해보자. 변하는 부분을 추상 메소드로 정의하여, 상속을 통해 구현한다. 

```java
abstract PreparedStatement makeStatement(Connection c) throws SQLException;

public class UserDaoDeleteAll extends UserDao{

    protected PreparedStatement makeStatement(Connection c) throws SQLException{
        PreparedStatement ps = c.prepareStatement("delete from users");
        return ps;
    }
}

```

 - 템플릿 메소드 패턴으로의 접근은 제한이 많다. 가장 큰 문제는 DAO 로직마다 상속을 통해 새로운 클래스를 만들어야 한다는 점이다. 또 확장구조가 이미 클래스를 설계하는 시점에서 고정되어 버린다는 점이다. 서브클래스들이 이미 클래스 레벨에서 컴파일 시점에 이미 그 관계가 결정되어 있다. 
 - 개방폐쇄원칙OCP를 잘 지키는 구조이면서도 템플릿 메소드 패턴보다 유연하고 확장성이 뛰어난 것이, 오브젝트를 아예 둘로 분리하고 클래스 레벨에서는 인터페이스를 통해서만 의존하다록 만다는 전략패턴을 적용하자. 

 ```java
 public interface StatementStrategy {
    PreparedStatement makePreparedStatement(Connection c) throws SQLException;
}

public class DeleteAllStatement implements StatementStrategy{
    @Override
    public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
        return c.prepareStatement("delete from users");
    }
}
 ```

 - 아래와 같이 DeleteAllStatement을 적용해보았다, 하지만 구체클래스를 사용하도록 고정한 것이 좀 이상하다. 전략패턴에 따르면 Context 가 어떤 전략을 사용하게 할 것인가는 Context 를 사용하는 앞단의 Client 가 결정하는게 일반적이다. 
 
 ```java
public void deleteAll() throws SQLException {
        Connection c = null;
        PreparedStatement ps = null;

        try{
            c = connectionMaker.getConnection();
            StatementStrategy strategy = new DeleteAllStatement();
            PreparedStatement ps = strategy.makePreparedStatement(c);
            ps.executeUpdate();

        }catch(SQLException e){
            throw e;
        }finally{
            if(ps!=null) ps.close();
            if(c!=null) c.close();
        }
    }
 ```

 - 전략패턴에서는 클라이언트에서 전략을 넣어주어야 한다. `StatementStrategy strategy = new DeleteAllStatement()` 이 부분은 클라이언트에 들어가야 할 코드이다. 이를 위해 전략 인터페이스인 StatementStrategy 를 파라미터로 지정할 필요가 있다. 
 - 컨텍스트에 해당하는(변하지 않는 부분) 부분을 별도의 메소드로 추출하고, 
 - 변하는 부분을 파라미터로 지정하였다. 

 ```java
 public void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException {
        Connection c = null;
        PreparedStatement ps = null;

        try{
            c = connectionMaker.getConnection();
            ps = stmt.makePreparedStatement(c);
            ps.executeUpdate();

        }catch(SQLException e){
            throw e;
        }finally{
            if(ps!=null) ps.close();
            if(c!=null) c.close();
        }
    }
 ```

  -  모든 JDBC 코드의 틀에 박힌 작업이 이 컨텍스트 메소드 안에 잘 담겨 있다. 
  - 다음은 클라이언트 에서 호출하는 부분이다. 

```java
    public void deleteAll() throws SQLException {
        jdbcContextWithStatementStrategy(new DeleteAllStatement());
    }
```

  - 아래 add 메소드도 변하는 부분을 전략으로 분리해보자.

```java
public void add(User user) throws ClassNotFoundException, SQLException {
        Class.forName("org.h2.Driver");
        Connection c = connectionMaker.getConnection();

        PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }
```  

 - 앗, user 를 찾을수 없다는 컴파일 오류가 발생

```java
public class AddStatement implements StatementStrategy{
    @Override
    public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
        PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        return null;
    }
}
```

 - 클라이언트에서 User 타입 오브젝트를 받을 수 있도록 생성자롤 만든다. 

```java
public class AddStatement implements StatementStrategy{
    User user;

    public AddStatement(User user){
        this.user = user;
    }
```

- 클라이언트에서 add() 기능을 호출

```java
public void addUser(User user) throws SQLException {
        jdbcContextWithStatementStrategy(new AddStatement(user));
    }
```

- 현재 구조도 약간 문제가 있다. DAO메서드마다 새로운 StatementStrategy 구현 클래스를 만들어야 한다. 이렇게 되면 클래스 개수가 많이 늘어나게 된다. 또다른 불만은 DAO메서드에서 StatementStreategy 에 전달할 User 와 같은 부가적인 정보가 있는 경우, 이를 위해 오브젝트를 전달받는 생성자와 이를 저장해둘 인스턴스 변수를 번거롭게 만들어야 한다는 점이다. 

- 클래스 파일이 많아지는 문제는 간단한 해결책이 있다. 로컬 클래스를 사용하는 것이다. 

```java
    public void addUser(User user) throws SQLException {
        class AddStatement implements StatementStrategy{
            User user;

            public AddStatement(User user){
                this.user = user;
            }

            @Override
            public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
                PreparedStatement ps = c.prepareStatement(
                        "insert into users(id, name, password) values(?,?,?)");
                ps.setString(1, user.getId());
                ps.setString(2, user.getName());
                ps.setString(3, user.getPassword());

                return null;
            }
        }
        jdbcContextWithStatementStrategy(new AddStatement(user));
    }

```

 - AddStatement 가 사용될 곳이 add() 메소드 뿐이라면, 이렇게 사용하기 전에 바로 정의해서 쓰는것도 나쁘지 않다. 로컬 클래스는 클래스가 내부 클래스이기 때문에 자신이 선언된 곳의 정보에 접근할 수 있다. 
 - AddStatement 내에 생성자로 user 를 받을 필요가 없다. 내부 메소드는 자신이 정의된 메소드의 로컬 변수에 직접 접근할 수 있기 때문이다. 다만 내부클래스에서 외부의 변수를 사용할때 외부 변수는 판드시 final 로 선언해줘야 한다. 
 - 

```java
    public void addUser(final User user) throws SQLException {
        class AddStatement implements StatementStrategy{
            @Override
            public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
                PreparedStatement ps = c.prepareStatement(
                        "insert into users(id, name, password) values(?,?,?)");
                //로컬클래스에서 외부 메소드 로컬변수에 직접 접근 가능
                ps.setString(1, user.getId());
                ps.setString(2, user.getName());
                ps.setString(3, user.getPassword());

                return null;
            }
        }
        //생성자로 user 를전달할 필요 없음
        jdbcContextWithStatementStrategy(new AddStatement());
    }
``` 

 - 이번에는 AddStatement 를 익명클래스로 만들어 좀 더 간결하게 해보자.
 - 이름이 없기 때문에 클래스 자신의 타입을 가질 수 없고, 구현한 인터페이스 타입의 변수에만 저장할 수 있다.
 
```java
public void addUser(final User user) throws SQLException {

        StatementStrategy st = new StatementStrategy() {
            @Override
            public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
                PreparedStatement ps = c.prepareStatement(
                        "insert into users(id, name, password) values(?,?,?)");
                //로컬클래스에서 외부 메소드 로컬변수에 직접 접근 가능
                ps.setString(1, user.getId());
                ps.setString(2, user.getName());
                ps.setString(3, user.getPassword());
                return ps;
            }
        };

        //생성자로 user 를전달할 필요 없음
        jdbcContextWithStatementStrategy(st);
    }
```

- 익명내부 클래스의 오브젝트는 딱 한번만 사용할테니 굳이 변수에 담아둘 필요가 없다. 아래와 같이 파라미터에서 바로 생성하는 편이 낫다. 

```java
    public void addUser(final User user) throws SQLException {
        //생성자로 user 를전달할 필요 없음
        jdbcContextWithStatementStrategy(new StatementStrategy() {
            @Override
            public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
                PreparedStatement ps = c.prepareStatement(
                        "insert into users(id, name, password) values(?,?,?)");
                //로컬클래스에서 외부 메소드 로컬변수에 직접 접근 가능
                ps.setString(1, user.getId());
                ps.setString(2, user.getName());
                ps.setString(3, user.getPassword());
                return ps;
            }
        });
    }
```

- 마찬가지로 deleteAll()도 동일하게 해준다. 

```java
    public void deleteAll() throws SQLException {
        jdbcContextWithStatementStrategy(new DeleteAllStatement());
    }

    public void deleteAll() throws SQLException {
        jdbcContextWithStatementStrategy(new StatementStrategy() {
            @Override
            public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
                return c.prepareStatement("delete from users");
            }
        });
    }
```

 - jdbcContextWithStatementStrategy 는 다른 DAO에서도 사용가능하다. UserDAO 클래스 밖으로 독립시켜 다른 DAO가 사용할 수 있게 해보자.

 - JdbcContext 를 만들어 컨텍스트 역할을 하는 메서드를 가져왔다. 그런데 connection을 가져올 수 있는 DataSource 가 필요하다. 멤버변수로 선언하고, 주입받을수 있게 set메서드를 만든다. 

```java
public class JdbcContext {

    private DataSource dataSource;

    public void setDataSource(DataSource dataSource){
        this.dataSource = dataSource;
    }

    public void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException {
        Connection c = null;
        PreparedStatement ps = null;

        try{
            c = dataSource.getConnection();
            ps = stmt.makePreparedStatement(c);
            ps.executeUpdate();

        }catch(SQLException e){
            throw e;
        }finally{
            if(ps!=null) ps.close();
            if(c!=null) c.close();
        }
    }
}
```

 - 다음은 UserDao에서 jdbcContextWithStatementStrategy를 삭제하고, JdbcContext를 멤버변수로 두어, 사용한다. 

```java
public class UserDao {

    private JdbcContext jdbcContext;

    public void setJdbcContext(JdbcContext jdbcContext){
        this.jdbcContext = jdbcContext;
    }

    public void deleteAll() throws SQLException {
        jdbcContext.jdbcContextWithStatementStrategy(new StatementStrategy() {
            @Override
            public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
                return c.prepareStatement("delete from users");
            }
        });
    }
```

 - UserDao는 이제 JdbcContext 에 의존한다. 그런데 jdbcContext 는 구체 클래스이다. 스프링은 기본적으로 인터페이스를 사이에 두고 의존 클래스를 바꿔서 사용하도록 하는것이 목적이다. 하지만 jdbcContext 는 jdbcContext를 제공해주는 서비스 오브젝트로서의 의미가 있을뿐 구현방법이 바뀔 가능성은 없다.

 - 의존관계주입(DI)이라는 개념을 충실히 따르자면, 인터페이스를 사이에 둬서 클래스 레벨에서는 의존관계가 고정되지 않게 하고, 런타임시에 의존할 오브젝트와 관계를 다이내믹하게 주입해주는 것이 맞다. 그러나 스프링DI는 넓게 보자면 객체의 생성과 관계 설정에 대한 제어 권한을 오브젝트에서 제거하고 외부로 위임했다는  IoC라는 개념을 포괄한다. 

- UserDao와 JdbcContext 는 매우 긴밀한 관계를 가지고 강하게 결합되어 있다. UserDao 는 항상 JdbcContext 와 사용돼야 한다.하지만 인터페이스를 써도 무관하다.

## 템플릿과 콜백

 - 지금까지 만든 코드는 전략패턴의 기본 구조에 익명 내부 클래스를 활용한 방식이다. 이런 방식을 템플릿 콜백 패턴이라고 한다. 
 - 템플릿은 고정된 작업흐름을 가진 코드를 재사용한다는 의미이다. 콜백은 템플릿 안에서 호출되는 것을 목적으로 만들어진 오브젝트를 말한다. 

 - 클라이언트의 역할은 템플릿 안에서 실행될 로직을 담은 콜백 오브젝트를 만들고, 콜백이 참조할 정보를 제공. 콜백은 클라이언트가 템플릿의 메소드를 호출할 때 파라미터로 전달한다.
 - 템플릿은 정해진 작업흐름을 따라 진행하다, 내부에서 생성한 참조 정보를 가지고 콜백 오브젝트 메소드를 호출한다. 콜백은 클라이언트 메소드에 있는 정보와 템플릿이 제공한 참조 정보를 이용해 작업수행, 그 결과를 다시 템플릿에 돌려준다. 
 - 템플릿은 콜백이 돌려준 정보를 사용해 작업을 마저 수행한다. 

 ## 콜백의 분리와 재활용

 - 콜백안에서도 변하는 부분과 변하지 않는 부분이 있다. 
 - 쿼리는 변하는 부분, 나머지는 변하지 않는 부분

 ```java
 public void deleteAll() throws SQLException {
        executeSql("delete from users");
    }

    private void executeSql(final String query) throws SQLException {
        jdbcContext.jdbcContextWithStatementStrategy(new StatementStrategy() {
            @Override
            public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
                return c.prepareStatement(query);
            }
        });
    }
 ```

  - 이제 DAO 메소드는 deleteAll() 메소드처럼 executeSql()을 호출하는 한줄이면 끝이다. 
  - 하지만 executeSql() 메소드를 UserDao에서만 사용하기는 아깝다. JdbcContext 로 옮겨보자. 그럼 아래와 같이 jdbcContext 에 있는 executeSql()을 호출하는 형태가 된다. 

```java
    public void deleteAll() throws SQLException {
        jdbcContext.executeSql("delete from users");
    }
```  

 - 다음은 템플릿 콜백 패턴을 연습.
 - 파일안에 숫자의 합을 모두 더하는 프로그램 작성.

 ```java
 @Slf4j
public class Calculator {
    public Integer calcSum(String filepath) throws IOException {
        BufferedReader br = null;
        try{
            br = new BufferedReader(new FileReader(filepath));
            Integer sum = 0;
            String line = null;
            while ((line = br.readLine()) != null) {
                sum += Integer.valueOf(line);
            }
            return sum;
        }catch(IOException e){
            log.error(e.getMessage());
            throw e;
        }finally {
            if(br != null) br.close();
        }
    }
}
 ```

 - 갑자기 모든 숫자의 곱을 계산하는 기능을 추가해야 한다고 하면?
 - 템플릿/콜백을 적용, 템플릿이 콜백에게 전달해줄 내부의 정보가 뭐고, 콜백이 템플릿에게 전달해줘야 할게 뭔지,,가장 쉽게 생각할 수 있는 구조는
 - 잘 모르겠어서,, 일단은 변하는 부분을 메소드로 추출

```java
@Slf4j
public class Calculator {
    public Integer calcSum(String filepath) throws IOException {
        BufferedReader br = null;
        try{
            br = new BufferedReader(new FileReader(filepath));
            return doSomethingWithReader(br);
        }catch(IOException e){
            log.error(e.getMessage());
            throw e;
        }finally {
            if(br != null) br.close();
        }
    }

    private Integer doSomethingWithReader(BufferedReader br) throws IOException {
        Integer sum = 0;
        String line = null;
        while ((line = br.readLine()) != null) {
            sum += Integer.valueOf(line);
        }
        return sum;
    }
}
```

- 추출된 부분을 콜백 인터페이스로 만들어보자.

```java
public interface BufferedReaderCallback {
    Integer doSomethingWithReader(BufferedReader br) throws IOException;
}
```

 -  calSum에 콜백메서드를 파라미터를 넣어주고, 호출하게 변경한다. 
 - calSum 이름도 범용적으로 fileReadTemplate 으로 바꿔준다. 

```java
@Slf4j
public class Calculator {
    public Integer fileReadTemplate(String filepath, BufferedReaderCallback callback) throws IOException {
        BufferedReader br = null;
        try{
            br = new BufferedReader(new FileReader(filepath));
            return callback.doSomethingWithReader(br);
        }catch(IOException e){
            log.error(e.getMessage());
            throw e;
        }finally {
            if(br != null) br.close();
        }
    }
}
```

 - 아래는 테스트 코드이다. 

 ```java
 @Slf4j
class CalculatorTest {

    @Test
    void sumOfNumbers() throws IOException {

        log.info("test start");

        Calculator calculator = new Calculator();
        String path = Objects.requireNonNull(
                getClass().getResource("numbers.txt")).getPath();

        BufferedReaderCallback callback = new BufferedReaderCallback() {
            @Override
            public Integer doSomethingWithReader(BufferedReader br) throws IOException {
                Integer sum = 0;
                String line = null;
                while ((line = br.readLine()) != null) {
                    sum += Integer.valueOf(line);
                }
                return sum;
            }
        };

        int sum = calculator.fileReadTemplate(path, callback);
        log.info("test end");
        Assertions.assertThat(sum).isEqualTo(15);
    }
}
 ```

  - Calculator 에 calSum 메서드를 추가한다. 

```java
    public Integer calSum(String filepath) throws IOException {
        BufferedReaderCallback callback = new BufferedReaderCallback() {
            @Override
            public Integer doSomethingWithReader(BufferedReader br) throws IOException {
                Integer sum = 0;
                String line = null;
                while ((line = br.readLine()) != null) {
                    sum += Integer.valueOf(line);
                }
                return sum;
            }
        };
        return fileReadTemplate(filepath, callback);
    }
``` 

- 테스트 코드에서 calSum메서드로 테스트 해본다. 

```java
@Test
    void sumOfNumbers() throws IOException {
        log.info("test start");
        Calculator calculator = new Calculator();
        String path = Objects.requireNonNull(
                getClass().getResource("numbers.txt")).getPath();

        int sum = calculator.calSum(path);
        log.info("test end");
        Assertions.assertThat(sum).isEqualTo(15);
    }
```

 - 이제 곱하기를 구현한다. 

```java
public Integer calMultiply(String filepath) throws IOException {
        BufferedReaderCallback callback = new BufferedReaderCallback() {
            @Override
            public Integer doSomethingWithReader(BufferedReader br) throws IOException {
                int multiplay = 1;
                String line = null;
                while ((line = br.readLine()) != null) {
                    multiplay *= Integer.parseInt(line);
                }
                return multiplay;
            }
        };
        return fileReadTemplate(filepath, callback);
    }
```

 - 더하기와 곱하기 로직이 중복되는 부분이 많이 보인다. 
 - 여기서 바뀌는 코드는 while문 안에 있는 계산 로직이다. 이 로직으로 전달되는 정보는 sum, multiplay 변수와 line 변수이다. 이걸 콜백으로 정의하면 아래와 같다. 

```java
 public interface LineCallBack {
    Integer doSomethingWithLine(String line, Integer value);
}
```

- 이제  LineCallBack 으로 바꿔보자. calSum과 calMultiply 메서드가 더 간결해졌다.

```java
 @Slf4j
public class Calculator {
    public Integer fileReadTemplate(String filepath, LineCallBack callback, Integer initVal) throws IOException {
        BufferedReader br = null;
        try{
            br = new BufferedReader(new FileReader(filepath));
            Integer res = initVal;
            String line = null;
            while ((line = br.readLine()) != null) {
                res = callback.doSomethingWithLine(line, res);
            }
            return res;
        }catch(IOException e){
            log.error(e.getMessage());
            throw e;
        }finally {
            if(br != null) br.close();
        }
    }

    public Integer calSum(String filepath) throws IOException {

        LineCallBack callback = new LineCallBack() {
            @Override
            public Integer doSomethingWithLine(String line, Integer value) {
                return Integer.parseInt(line) + value;
            }
        };
        return fileReadTemplate(filepath, callback, 0);

    }

    public Integer calMultiply(String filepath) throws IOException {
        LineCallBack callback = new LineCallBack() {
            @Override
            public Integer doSomethingWithLine(String line, Integer value) {
                return Integer.parseInt(line) * value;
            }
        };
        return fileReadTemplate(filepath, callback, 1);
    }

}
```

 - Integer 타입으로 고정되어 있는 것을 제네릭스를 사용하여 바꿔본다.

```java
public interface LineCallBack<T> {
    T doSomethingWithLine(String line, T value);
}
```

 - 아래와 같이 바꾼다.

```java
public <T> T fileReadTemplate(String filepath, LineCallBack<T> callback, T initVal) throws IOException {
        BufferedReader br = null;
        try{
            br = new BufferedReader(new FileReader(filepath));
            T res = initVal;
            String line = null;
            while ((line = br.readLine()) != null) {
                res = callback.doSomethingWithLine(line, res);
            }
            return res;
        }catch(IOException e){
            log.error(e.getMessage());
            throw e;
        }finally {
            if(br != null) br.close();
        }
    }

    public Integer calSum(String filepath) throws IOException {

        LineCallBack<Integer> callback = new LineCallBack<>() {
            @Override
            public Integer doSomethingWithLine(String line, Integer value) {
                return Integer.parseInt(line) + value;
            }
        };
        return fileReadTemplate(filepath, callback, 0);
    }


```

 - 새로운 기능을 하나 추가해보자, 파일안에 있는 숫자들을 일렬로 연결시켜보자

```java
    public String concatenate(String filepath) throws IOException {
        LineCallBack<String> callback = new LineCallBack<>() {
            @Override
            public String doSomethingWithLine(String line, String value) {
                return value + line;
            }
        };
        return fileReadTemplate(filepath, callback, "");
    }
```

## 스프링의 JdbcTemplate

 - JdbcContext 를 JdbcTemplate 으로 바꾼다. 

```java
public class UserDao {
    private JdbcTemplate jdbcTemplate;
    public void setJdbcContext(DataSource dataSource){
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public void deleteAll() throws SQLException {
        jdbcTemplate.update("delete from users");
    }

    public void add(User user) throws ClassNotFoundException, SQLException {
        jdbcTemplate.update("insert into users(id, name, password) values(?,?,?)",
                user.getId(), user.getName(), user.getPassword());
    }
```

 - getCount()도 만들어보자, 2개의 콜백이 등장하는데, 첫번째는 
 Connection 을 받고  PreparedStatement 를 돌려주는 거고, 두 번째는
 ResultSet 을 받고 결과를 돌려준다. 
 - 복잡해보이지만, 변하는 부분만 콜백으로 만들어 던져준다고 생각하면 이해가 된다

```java
public int getCount(){
        Integer result = 
        jdbcTemplate.query(new PreparedStatementCreator() {
                                               @Override
                                               public PreparedStatement createPreparedStatement(Connection conn) throws SQLException {
                                                   return conn.prepareStatement("select count(*) from users");
                                               }
                                           }, new ResultSetExtractor<Integer>() {
                                               @Override
                                               public Integer extractData(ResultSet rs) throws SQLException, DataAccessException {
                                                   rs.next();
                                                   return rs.getInt(1);
                                               }
                                           }
        );
        return result;
    }
```

 - 위 소스는 재사용되기 좋은 구조로, 다음과 같이 한줄로 표현할수 있다. 

 ```java
 public int getCount(){
        return jdbcTemplate.queryForObject("select count(*) from users", Integer.class);
    }
 ```

  - get 메서드는 복잡하다. 
  - 첫번째 파라미터는 PreparedStatement를 만들기 위한 SQL이고, 
  두번째는 여기 바인딩할 값들이다. 세번째는 실행결과를 매핑한다. 

```java
public User get(String id){
        return jdbcTemplate.queryForObject("select * form users where id=?",
                new Object[]{id},
                new RowMapper<User>() {
                    @Override
                    public User mapRow(ResultSet rs, int rowNum) throws SQLException {
                        User user = new User();
                        user.setId(rs.getString("id"));
                        user.setName(rs.getString("name"));
                        user.setPassword(rs.getString("password"));
                        return user;
                    }
                });
    }

 public List<User> getAll(){
        return jdbcTemplate.query("select * form users",
                new RowMapper<User>() {
                    @Override
                    public User mapRow(ResultSet rs, int rowNum) throws SQLException {
                        User user = new User();
                        user.setId(rs.getString("id"));
                        user.setName(rs.getString("name"));
                        user.setPassword(rs.getString("password"));
                        return user;
                    }
                });
    }
```

 - getAll 은 query 메서드를 사용한다. 만약 바인딩할 값이 있다면, 
 get 메서드처럼 두 번째 인자를 넣어주면 된다. 

## 재사용 가능한 콜백의 분리

 - 위 메서드를 보면 RowMapper가 중복으로 사용되었고, 추가로 사용될 여지가 있다. 재사용 가능하도록 독립시켜보자.

```java
    private RowMapper<User> userMapper =
            new RowMapper<User>() {
                @Override
                public User mapRow(ResultSet rs, int rowNum) throws SQLException {
                    User user = new User();
                    user.setId(rs.getString("id"));
                    user.setName(rs.getString("name"));
                    user.setPassword(rs.getString("password"));
                    return user;
                }
            };

    public User get(String id){
        return jdbcTemplate.queryForObject("slelect * form users where id=?",
                new Object[]{id},
                userMapper);
    }

    public List<User> getAll(){
        return jdbcTemplate.query("select * form users",
                userMapper);
    }
``` 

 - UserDao에는 User정보를 DB에 넣거나 가져오거나 조작하는 방법에 대한 핵심적인 로직만 담겨 있다. 반면 JDBC API를 사용하는 방식, 예외처리, 리소스 반납, DB연결을 어떻게 가져올지에 대한 책임과 관심은 모두 JdbcTemplate 에게 있다.
  - JdbcTemplate 은 DAO안에서 직접 만들어 사용하는계 스프링의 관례이지만 빈으로 등록하고  DI받아 사용할수도 있다. 





# Reference

> - 토비의 스프링 3.1 vol1 스프링의 이해와 원리
