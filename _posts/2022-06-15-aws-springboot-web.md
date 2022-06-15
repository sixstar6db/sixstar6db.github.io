---
title:  "[java] 스프링부트 웹서비스 AWS 구현"
excerpt: "스프링부트 웹서비스 AWS 구현"

categories: Java
tags:
  - [java]

toc: true
toc_sticky: true
 
date: 2022-06-15
last_modified_at: 2022-06-15
author_profile: false     
---

## 개요

 - 간단한 스프링 부트 웹서비스를 만들어 AWS에 올려본다. 이동욱저자의 `스프링부트와 AWS 혼자 구현하는 웹서비스`를 참고하였다. 완전히 똑같게 만들진 않고, 조금 다르게 해볼 생각이다. 예를 들어 책에서는 템플릿 엔진을 mustache 를 사용하였으나, 대신에 thymeleaf 로 해보려 한다. 

## 인텔리 제이 시작하기

> 메모리 설정

 - [Help - Change memory settings] 에서 메모리 설정이 가능하다. 적당한 메모리 사이즈를 모르겠다면, 인털리제이에서 설정된 값을 사용하는 것을 추천한다.

 - PC 메모리가 8G라면 1024~2048 , 16G라면 2048~4096을 선택해서 사용한다. (난 2048 설정)

![image1](/assets/images/page11/img1.png)

![image1](/assets/images/page11/img2.png)

## 스프링부트 프로젝트 생성하기

> 프로젝트 생성

![image1](/assets/images/page11/img3.png)

 - 지금까지는 메이븐만 주로 사용해왔는데, 한번 그레이들로 해보기로 함
 - build.gradle 을 스프링프로젝트로 구성하기 위한 설정을 해보았다. 

```java
buildscript {
    ext {
        springBootVersion = '2.1.7.RELEASE'
    }
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.sk'
version = '1.0-SNAPSHOT'
sourceCompatibility = 1.11

repositories {
    mavenCentral()
}

dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
``` 

 - 그레이들 reload를 하였는데 오류가 발생하였다. 
 `Could not find method compile() for arguments ....`
 - 뭔가 그레이들 버전이 맞지 않는다는 게 느껴졌다.  

![image1](/assets/images/page11/img4.png)

 - 저자의 github 에 보니, 해당 그레이들 설정은 4.10.2까지 정상작동 한다고 한다. 현업에서는 아직도 그레이들 4가 많이 사용되어서 저렇게 했다는,,

 - 방법은 그레이들을 다운그레이드 하여 진행하면 된다. 
 - 방법1. 은 gradle-wrapper.properties 파일의 버전 정보를 직접 수정하는 것이다. (gradle-4.10.2-bin.zip)
 - 방법2. 는 터미널에서 `gradlew wrapper --gradle-version 4.10.2` 명령어를 치면 된다고 하는데, 잘 되지않아 방법1을 사용했다. 
 
```java
buildscript {
    ext {
        springBootVersion = '2.1.7.RELEASE'
    }
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.sk'
version = '1.0-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

 - `io.spring.dependency-management` 는 스프링 부트의 의존성들을 관리해주는 플러그인이라 꼭 추가해주어야 한다.
 - 아래는 그레이들 5 이상으로 했을 경우이다. 나중에 배포까지 완료 한 후에 그레이들 5, 
 스프링부트도 2.7.0으로 업그레이드 해봐야 겠다. 

```java
plugins {
    id 'org.springframework.boot' version '2.7.0'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
}

group = 'com.sk'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

## 깃 연동

 - [shift + shift] 단축키를 누르고, 'share project on github' 를 입력한다. 
 - Repository name 은 프로젝트명과 동일하게 한 후, Add Count 를 클릭한다. 
 - github 로그인 페이지로 연결되어 인증하는 방식으로 진행된다. 

![image1](/assets/images/page11/img5.png) 

![image1](/assets/images/page11/img6.png) 

 - 앗, 그런데 git 이 설치되어 있지 않다.  git 설치를 진행한다. 

![image1](/assets/images/page11/img7.png)  

 - 설치 완료 후, 다시  'share project on github'를 진행하니, 이미 동일한 이름으로 레파지토리가 있다고 나온다. ㅜ 
 - 인텔리제이에서 github 연동하는 것을 구글링해보니, 레파지토리 생성을 인텔리제이에서 하는 경우와  github 에서 하는 2가지 방식에 대해 설명해놓은 것이 있어 참고하며 진행하였다. 

![image1](/assets/images/page11/img8.png)  

 - [shift + shift] 단축키를 이용해 `Create git repository` 를 입력한다.  
 - 대상 프로젝트 디렉토리를 선택하고, OK를 클릭. 

![image1](/assets/images/page11/img10.png)  

 - `git > add` 를 진행한다. commit 전 add를 통해 Staging에 코드를 추가하는 것이라고 한다. 
 - `git > commit directory` 를 진행한다. 변경사항을 확정하여 Local 영역에 저장하는 작업.

![image1](/assets/images/page11/img11.png)  
![image1](/assets/images/page11/img12.png)  

- .idea 에 있는 파일을 제외(인텔리제이에서 생성하는 파일들로 github에 올릴필요 없음)하고, 나머지 파일을 선택 한 후, `commit and push` 를 클릭한다.  

![image1](/assets/images/page11/img13.png) 

- 연동될 github 레파지토리 주소를 입력한다. 

![image1](/assets/images/page11/img14.png) 

- PUSH 클릭

![image1](/assets/images/page11/img15.png) 

- github 에 접속해보면 소스가 올라가 있음.

![image1](/assets/images/page11/img16.png) 

- git에서는 .gitignore 파일을 이용하여, 커밋대상제외를 관리한다. 하지만 인텔리제이에서는 .gitignore 파일에 대한 기본적인 지원이 없다. 대신 플러그인을 지원한다. 

- 플러그인은 `파일위치자동완성/이그노어처리여부 확인/다양한 이그노어 파일지원` 등의 기능을 지원한다. 

- [shift + shift] 클릭후에 plugins 를 검색한다.
- `.ignore` 를 검색후 설치 하고, 인텔리제이를 재시작한다. 

![image1](/assets/images/page11/img17.png) 

- 프로젝트에서 우클릭후 NEW-.ignore File에서 .gitignore File 을 생성한다. 
- Generator 화면는 이그노어 템플릿을 선택하는 화면인데, 만들어둔 것이 없으므로 그냥 Generate 를 클릭한다. 

![image1](/assets/images/page11/img18.png) 
![image1](/assets/images/page11/img19.png) 

## 테스트 코드 작성

 - 저자는 테스트 코드의 중요성을 강조함. 단위 테스트에 대한 이점은 다음과 같다. 
   1. 단위테스트는 개발단계 초기에 문제를 발견하게 도와준다. 
   2. 리팩토링 또는 라이브러리 업데이트시 기능이 올바르게 동작하는지 확인할 수 있다. 
   3. 불확실성 감소
   4. 단위테스트는 시스템에 대한 실제 문서를 제공한다. 단위테스트 자체를 문서로 사용할 수 있다.
 - TDD 배울수 있는 링크(채수원, TDD실천법과 도구) https://repo.yona.io/doortts/blog/issue/1

 - 간단한 HelloController 를 생성 후 테스트 코드를 검증한다. 

```java
@RestController
    class HelloController {

        @GetMapping("/hello")
        public String hello() {
            return "hello";
        }
    }
```

```java
package com.sk;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@WebMvcTest(controllers = App.HelloController.class)
public class HelloControllerTest {

    @Autowired
    private MockMvc mvc;

    @Test
    public void hello가_리턴된다() throws Exception{
        String hello = "hello";

        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
    }
}

```

 - 위 코드에 대한 간단한 설명 
 1. @RunWith(SpringRunner.class)
   : Junit 에 내장된 실행자외에 SpringRunner.class 라는 확장된 클래스를 실행하게 된다. @Autowired를 통해 빈을 주입받을 수 있거나 하는 것처럼 스프링 컨테이너를 활용할 수 있게 된다. JUnit5에서는 @RunWith 역할을 하는 어노테이션이 @ExtendWith 로 변경되었다. 그리고 해당 어노테이션은 @SpringBootTest 어노테이션에 포함되어있기 때문에 별도로 정의할 필요도 없다. 

 2. @WebMvcTest 
   : @Controler 등을 사용하의 Web(Spring MVC)를 테스트 할 수 있다. 단, @Service, @Component, @Repository 등은 사용X

 3. MockMVC
    : 웹API 테스트 할 때 사용

 4. mvc.perform(get("/hello")) : HTTP GET 요청
 5. andExpect(Status().isOk()) : perform 메서드의 결과를 검증한다. HTTP header header 의 status 를 검증한다. isOk()는 200 코드 인지 검증.
 6. andExpect(content().string(hello)) : 응답 본문을 검증한다. 

## 롬복 설치하기

 - build.gradle 에 의존성 추가 
   `compile('org.projectlombok:lombok)`
 - 롬복 플러그인 설치, [shift + shift] 단축키 입력 후, plugins 입력
 - 플러그인 검색창에서 lombok 을 입력후 검색한다. 예전에는 lombok 으로 조회가 되었는데, `kfang agent lombok` 이라는게 조회된다. 뭔가 기존 롬복의 향상된 버전으로 보인다. 일단 설치!

![image1](/assets/images/page11/img20.png)

 - 인텔리제이를 재시작하고, Enable annotation processing 을 체크한다. 
 - build.gradle 에 라이브러리를 추가하는 것과 Enable annotation processing 체크는 프로젝트마다 해주어야 한다. 플러그인 설치는 반복할필요 없다.
![image1](/assets/images/page11/img21.png)

- 롬복을 사용하여 HelloController를 바꿔보자

```java
@RestController
    class HelloController {

        @GetMapping("/hello")
        public String hello() {
            return "hello";
        }

        @GetMapping("/hello/dto")
        public HelloResponseDto helloDto(@RequestParam("name") String name,
                                         @RequestParam("amount") int amount){
            return new HelloResponseDto(name, amount);
        }
    }

    @Getter
    @RequiredArgsConstructor
    static class HelloResponseDto{
        private final String name;
        private final int amount;
    }
```

 - 아래는 테스트 코드이다. 

```java
@Test
    public void helloDto_정상리턴() throws Exception {
        String name = "hello";
        int amount = 1000;

        mvc.perform(get("/hello/dto")
                .param("name", name)
                .param("amount", String.valueOf(amount)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name", is(name)))
                .andExpect(jsonPath("$.amount", is(amount)));
    }
```

 - param : api 테스트 할 때 요청 파라미터 설정, 단 String 값만 허용되어 숫자/날짜 등의 데이터를 문자열로 변경해야 한다. 

 - jsonPath : jJSON 응답값을 필드별로 검증할 수 있는 메소드로, $를 기준으로 필드명 명시

## 그레이들 설정 변경

 - 스프링을 실행할때 gradle을 통해 실행될때가 있어 속도가 느릴수 있다. 아래 그림처럼 바꿔주면 인텔리제이 자체에서 실행되므로 속도가 개선된다. 

 ![image1](/assets/images/page11/img22.png)

## 스프링부트에서의 JPA

 - 객체지향적으로 프로그래밍을 하고, JPA가 이를 관계형 데이터베이스에 맞게 SQL을 대신 생성해서 실행해준다. 객체 중심으로 개발하니 생산성도 향상되고 유지보수도 쉬워진다. 이런점 때문에 점점 JPA가 표준 기술로 자리 잡고 있다.

> Spring Data JPA 

 - JPA는 인터페이스이고, 그 구현체로 Hibernate 가 있고, 그것을 더 쉽게 사용하고자 추상화 시킨 Spring DAta JPA가 있다. 
 - 장점 : 구현체 교체용이 / 저장소 교체 용이
 작, Hibernate 구현체를 쓰다 다른거로 교체하기도 용이하고, Spring data jpa 를 쓰다가 트래픽증가로  몽고 DB를 쓰기 위해 Spring Data MongoDB로 바꾸는것도 용이하다.

 - build.gradle 에 의존성을 추가한다. 
 - spring-boot-starter-data-jpa : 스프링부트 버전에 맞춰 자동으로 JPA관련 라이브러리들의 버전을 관리해준다. 
 - com.h2database:h2 : 인메모리 관계형 데이터베이스로 별도 설치 없이 의존성만으로 관리 가능하다. 테스트 용도로 많이 사용된다.

```java
dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.projectlombok:lombok')
    compile('org.springframework.boot:spring-boot-starter-data-jpa')
    compile('com.h2database:h2')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
``` 

 - 도메인 모델을 정의한다. Posts 클래스는 실제 테이블과 매칭될 것이며, 보통 Entity 클래스라 한다. 
 - @Entity : 테이블과 링크될 클래스를 나타낸다. 클래스의 이름과 테이블명은 다음과 같이 매칭된다. 
       ex : SalesManager.java -> SALES_MANAGER table
 - @Id : 테이블의 PK를 나타낸다. 
 - @GeneratedValue : PK의 생성 규칙을 나타낸다. 스프링부트 2.0에서는 `GenerationType.IDENTITY` 옵션을 추가해야 auto_increment 가 된다. 
 - @Column : 선언하지 않아도 컬럼이 되지만, 추가 변경이 필요한 옵션이 있는 경우 선언한다. 문자열의 경우 기본은 varchar(255)이며 변경 필요한 값으로 선언할수 있다. 타입을 TEXT로도 변경할 수 있다.



```java
@Getter
@NoArgsConstructor
@Entity
public class Posts {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;

    @Builder
    public Posts(String title, String content, String author){
        this.title = title;
        this.content = content;
        this.author = author;
    }
}
```

> PK키는  Long 타입의 Auto_increment를 추천

  - 주민번호 같은 유니크키나 복합키는 FK를 맺을때 다른 테이블에서 복합키 전부를 갖고 있거나, 중간 테이블을 하나 둬야 하는 상황이 발생할 수 있다.
  - 인덱스에 좋지 않은 영향을 끼친다고 함
  - 유니크한 조건이 변경될때 PK전체를 수정해야 하는 일 발생
  - 주민번호, 복합키등 필요하면 유니크 키로 별도로 추가

> 클래스 명과 가까운 곳에 @Entity를 선언하고 먼 곳에 롬복 관련 어노테이션을 선언한다. 

  - 롬복이 필요없는 환경에서 쉽게 제거 가능하다. 

> Entity 클래스에는 Setter 메소드를 만들지 않는다. 

  - 값 변경 필요시, 명확히 목적과 의도를 드러내는 메소드를 선언.

> Repository 클래스를 생성한다. 

```java
package com.sk.started.posts;

import org.springframework.data.jpa.repository.JpaRepository;

public interface PostsRepository extends JpaRepository<Posts, Long> {
}
```  

 - `JpaRepository<Posts, Long>` 를 상속하면 자동으로 CRUD 메소드를 만들어준다. 
 - @Repository를 추가할 필요 없다. 대신 Posts 와 PostsRepository 는 패키지상 함께 위치해 있어야 한다. Entity는 기본 Repository 없이는 제대로 역할 할수 없다. 

# Reference
> - 스프링부트와 AWS 혼자 구현하는 웹서비스 ( by 이동욱)

- 참고 사이트 
https://gocheat.github.io/common/clean-arc-package/