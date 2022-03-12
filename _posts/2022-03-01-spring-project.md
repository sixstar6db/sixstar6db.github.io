---
title:  "[SPRING] 스프링 프로젝트 생성하기(mybatis-jsp-oracle)"
excerpt: "스프링 프로젝트 템플릿"

categories: Java
tags:
  - [spring]

toc: true
toc_sticky: true
 
date: 2022-03-01
last_modified_at: 2022-03-01
author_profile: false     
---

## java maven project 생성

 -  스프링부트를 사용하지 않고, 스프링프레임워크 + 톰캣 개발 환경 셋팅

![image1](/assets/images/page7/step1.JPG)
![image1](/assets/images/page7/step2.JPG)
![image1](/assets/images/page7/step3.JPG)

- pom.xml 에 의존관계 라이브러리 설정

```xml

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>chn.spring</groupId>
  <artifactId>chn_spring_project</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>war</packaging>

  <properties>
	<java-version>1.8</java-version>
	<org.aspectj-version>1.6.10</org.aspectj-version>
	<org.slf4j-version>2.11.2</org.slf4j-version>
	<logback.version>1.2.3</logback.version>
	<maven.test.skip>true</maven.test.skip>

	<spring.version>4.3.22.RELEASE</spring.version>
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>

<dependencies>
<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>servlet-api</artifactId>
			<version>2.5</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jsp-api</artifactId>
			<version>2.0</version>
			<scope>provided</scope>
		</dependency>
		<!-- Jackson JSON Processor -->
		<dependency>
			<groupId>org.codehaus.jackson</groupId>
			<artifactId>jackson-mapper-asl</artifactId>
			<version>1.9.13</version>
		</dependency>

		<dependency>
			<groupId>commons-fileupload</groupId>
			<artifactId>commons-fileupload</artifactId>
			<version>1.3</version>
		</dependency>
		<!-- Logging -->
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>jcl-over-slf4j</artifactId>
			<version>1.7.26</version>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.7.26</version>
			<scope>runtime</scope>
		</dependency>

		<!-- 2016-05 추가 -joal -->
		<dependency>
			<groupId>ch.qos.logback</groupId>
			<artifactId>logback-classic</artifactId>
			<version>${logback.version}</version>
		</dependency>
		<dependency>
			<groupId>org.logback-extensions</groupId>
			<artifactId>logback-ext-spring</artifactId>
			<version>0.1.4</version>
		</dependency>
		
		<!-- Spring -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${spring.version}</version>
			<!-- <systemPath>${project.basedir}/extLib/spring-webmvc-4.3.16.RELEASE.jar</systemPath> -->
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>${spring.version}</version>
			<!--<systemPath>${project.basedir}/extLib/spring-jdbc-4.3.16.RELEASE.jar</systemPath> -->
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>4.2.4.RELEASE</version>
			<scope>test</scope>
		</dependency>
		<!--JDK1.5 available -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>3.4.6</version>
		</dependency>
		<!--Spring Mybatis available -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis-spring</artifactId>
			<version>2.0.1</version>
		</dependency>
		
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>1.7.26</version>
		</dependency>
		<dependency>
			<groupId>commons-dbcp</groupId>
			<artifactId>commons-dbcp</artifactId>
			<version>1.4</version>
		</dependency>

		<!-- jstl 추가 2016.04.26 -->
		<dependency>
			<groupId>jstl</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>

</dependencies>

<build>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-clean-plugin</artifactId>
			<configuration>
				<filesets>
					<fileset>
						<directory>WEB-INF/lib</directory>
					</fileset>
				</filesets>
			</configuration>
		</plugin>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-compiler-plugin</artifactId>
			<version>3.1</version>
			<configuration>
				<source>${java-version}</source>
				<target>${java-version}</target>
				<showWarnings>true</showWarnings>
				<showDeprecation>true</showDeprecation>
			</configuration>
		</plugin>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-war-plugin</artifactId>
			<configuration>
				<webResources>
					<resource>
						<directory>${build.sourceDirectory}</directory>
						<targetPath>WEB-INF/classes</targetPath>
					</resource>
				</webResources>
			</configuration>
		</plugin>
	</plugins>
</build>

</project>
```

 - jdk 8 로 변경 후에 maven update 를 해준다. 

![image1](/assets/images/page7/step4.JPG)

 - src/webapp/WEB-INF/web.xml 파일을 생성한다. 

 - web.xml 내용

 ```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>
			/WEB-INF/context/applicationContext.xml
		</param-value>
	</context-param>
	
	<filter>
   		<filter-name>CharacterEncodingFilter</filter-name>
    	<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    	<init-param>
      		<param-name>encoding</param-name>
      		<param-value>UTF-8</param-value>
    	</init-param>
    	<init-param>
      		<param-name>forceEncoding</param-name>
      		<param-value>true</param-value>
    	</init-param>
  	</filter>

  	<filter-mapping>
    	<filter-name>CharacterEncodingFilter</filter-name>
    	<url-pattern>/*</url-pattern>
  	</filter-mapping>

	<servlet>
		<servlet-name>integrated-log</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>
                /WEB-INF/context/integratedLogServlet.xml
            </param-value>
		</init-param>
		<load-on-startup>2</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>integrated-log</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
	
	<session-config> 
           <session-timeout>60</session-timeout> 
    </session-config> 
    
</web-app>


 ```

 - ContextLoaderListener(applicationContext.xml)은 비즈니스 로직을 정의한 스프링설정 파일이다. DispatcherServlet보다 먼저 로드한다. 
 - DispatcherServlet(integratedLogServlet.xml)은 ContextLoaderListener 에서 생성하는 빈을 참조하여 MVC를 구동한다.

- /WEB-INF/context/integratedLogServlet.xml 파일을 생성한다. 
- 스프링프레임워크 MVC를 위한 기본 설정 정보가 들어간다. 
- mvc:annotation-driven 은 스프링mvc에 필요한 컴포넌트들을 빈으로 등록한다.
- mvc:resources 은 정적자원 서비스를 위한 매핑 설정이다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns:beans="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:task="http://www.springframework.org/schema/task"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
						http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
						http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
						http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
						http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
						http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task.xsd   ">
	<context:component-scan base-package="com.sk"/>
	<!-- 기본 mvc 설정 -->
	<mvc:annotation-driven />
	
	<!-- jsp 설정 -->
	<beans:bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<beans:property name="prefix" value="/WEB-INF/view/" />
		<beans:property name="suffix" value=".jsp" />
	</beans:bean>
	
	<!-- 정적 자원 -->
	<mvc:resources mapping="/**" location="/resources/" />
	
</beans:beans>

```

 - applicationContext.xml은 일단 빈 파일로 생성해둔다. 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns:beans="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:task="http://www.springframework.org/schema/task"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
						http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
						http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
						http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
						http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
						http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task.xsd   ">
						
</beans:beans>

``` 

 - 웹어플리케이션을 배포할 톰캣 서버를 등록한다. 

![image1](/assets/images/page7/step6.JPG)
![image1](/assets/images/page7/step7.JPG)
![image1](/assets/images/page7/step8.JPG)

 - context path 경로를 설정한다. Server.xml 에 Context 요소의 path 속성을 '/' 로 변경

 - 간단한 HelloController 를 작성한다. 

```java
package com.sk.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
	
	@GetMapping("/hello")
	public String hello() {
		return "hello";
	}
}
```

 - 톰캣을 구동하여 서비스가 잘 올라가는지 확인한다. 
 - http://localhost:8080/hello


## mybatis 연동

 - 데이터 도메인을 정의한다. 

 ```sql
 CREATE TABLE DONGBUWEB.TMP_USER
(
    U_NO    NUMBER(11) NOT NULL,
    U_NAME  VARCHAR2(60),
    U_JUMIN VARCHAR2(4000),
    U_EMAIL VARCHAR2(100),
    U_DATE  DATE DEFAULT sysdate,
    U_PAY   NUMBER(7,2)
)
 ```

 ```java
 package com.sk.user;

import java.math.BigDecimal;
import java.util.Date;

public class User {
	
	private BigDecimal no;
	private String name;
	private String jumin;
	private String email;
	private Date date;
	private Integer pay;
	
	public BigDecimal getNo() {
		return no;
	}
	public void setNo(BigDecimal no) {
		this.no = no;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getJumin() {
		return jumin;
	}
	public void setJumin(String jumin) {
		this.jumin = jumin;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	public Date getDate() {
		return date;
	}
	public void setDate(Date date) {
		this.date = date;
	}
	public Integer getPay() {
		return pay;
	}
	public void setPay(Integer pay) {
		this.pay = pay;
	}
	@Override
	public String toString() {
		return "User [no=" + no + ", name=" + name + ", jumin=" + jumin + ", email=" + email + ", date=" + date
				+ ", pay=" + pay + "]";
	}
}
 ```

 - Mapper 인터페이스를 정의한다. 

 ```java
 package com.sk.user;

import java.util.List;

import org.springframework.stereotype.Repository;

@Repository
public interface UserMapper {
	
	public User findByNo(Integer no);
	
	public List<User> findAll();
	
	public void put(User user);

}

 ```

 -- 비즈니스 로직이 구현될 Service를 생성한다. 

```java
package com.sk.user;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserService {
	
	@Autowired UserMapper userMapper;
	
	public User findByNo(Integer no) {
		return userMapper.findByNo(no);
	}
	
	public List<User> findAll() {
		return userMapper.findAll();
	}
	
	public void putUser(User user) {
		userMapper.put(user);
	}

}

```

- 웹 요청을 받아서 처리할 컨트롤러를 구현한다. 

```java
package com.sk.user;

import java.io.IOException;
import java.util.List;

import org.codehaus.jackson.map.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequestMapping("user")
public class UserController {
	
	private final Logger log = LoggerFactory.getLogger(getClass());
	private ObjectMapper objectMapper = new ObjectMapper();
	
	@Autowired UserService userService;
	
	/**
	 *  request : Get @Pathvariable
	 *  response : view
	 */
	@GetMapping("get/{no}")
	public String getUserPut(@PathVariable("no") Integer number, Model model) {
		User user = userService.findByNo(number);
		model.addAttribute("user", user);
		return "jsp/user";
	}
	
	/*
	 * request : GET
	 * response : view
	 */
	@GetMapping("put")
	public String getUserPut() {
		return "jsp/put";
	}
	
	/**
	 * request : POST form parameter
	 * response : view redirect
	 */
	@PostMapping("put")
	public String userPut(User user) {
		userService.putUser(user);
		return "redirect:/user/list";
	}
	
	/**
	 * request : GET
	 * response : view 
	 */
	@GetMapping("list")
	public String userList(Model model) {
		List<User> users = userService.findAll();
		model.addAttribute("users", users);
		return "jsp/list";
	}
	
	/**
	 * Ajax 요청
	 *   - reuest : parameter
	 *   - response : json
	 */
	@ResponseBody
	@RequestMapping("putAjax")
	public User userPutJson2(@ModelAttribute User user) throws IOException {
		log.info("request queryString,  user={}", user);
		userService.putUser(user);
		return user;
	}
	
	/*
	 * Ajax 요청
	 *    - request : json
	 *    - response: json
	 */
	@ResponseBody
	@RequestMapping("putAjaxJson")
	public User userPutJson3(@RequestBody String userJson) throws IOException {
		User user = objectMapper.readValue(userJson, User.class);
		log.info("request json, user={}", user);
		userService.putUser(user);
		return user;
	}
	
	/**
	 * Get Home 페이지
	 */
	@GetMapping("listClientRendering")
	public String list2() {
		return "jsp/list2";
	}
	
	/*
	 * Ajax 요청
	 *    - response: json
	 */
	@ResponseBody
	@PostMapping("listJson")
	public List<User> userListJson() throws IOException {
		return userService.findAll();
	}
	
}

```

 - /WEB-INF/sqlmap 하위에 UserMapper.xml 파일을 만들어 매핑 쿼리를 작성한다. 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sk.user.UserMapper">
	<select id="findAll" resultType="com.sk.user.User">
		SELECT
			 U_NO as "no"
			,U_NAME as "name"
			,U_JUMIN as "jumin"
			,U_EMAIL as "email"
			,U_DATE as "date"
			,U_PAY as "pay"
		FROM TMP_USER
		ORDER BY U_NO DESC
	</select>
	
	<select id="findByNo" parameterType="Integer" resultType="com.sk.user.User">
		SELECT
			 U_NO as "no"
			,U_NAME as "name"
			,U_JUMIN as "jumin"
			,U_EMAIL as "email"
			,U_DATE as "date"
			,U_PAY as "pay"
		FROM TMP_USER
		WHERE U_NO = #{no}
	</select>
	
	<insert id="put" parameterType="com.sk.user.User">
		insert into TMP_USER (U_NO, U_NAME, U_JUMIN, U_EMAIL, U_DATE, U_PAY) 
			values (TMP_USER_SEQ.NEXTVAL, #{name}, #{jumin}, #{email}, SYSDATE, #{pay})
	</insert>
	
</mapper>
``` 

- /WEB-INF/context/mybatis-context.xml 파일을 생성 후, 설정 정보를 작성한다. 

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:jee="http://www.springframework.org/schema/jee"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
						http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
						http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
						http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee.xsd
						http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring.xsd
						">
	<!-- jndi 정보 연결 -->					
	<jee:jndi-lookup id="dataSource"
	    jndi-name="jdbc/home_ds"
	    cache="true"
	    resource-ref="true"
	    lookup-on-startup="false"
	    proxy-interface="javax.sql.DataSource" /> 
	<!-- 
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="oracle.jdbc.driver.OracleDriver" />
		<property name="url" value="jdbc:oracle:thin:@172.20.14.206:1521:HOMEDEV" />
		<property name="username" value="dongbuweb" />
		<property name="password" value="dongbuweb" />
	</bean>     -->

	<bean id="sqlSessionFactoryDefault" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="mapperLocations" value="WEB-INF/sqlmap/*.xml" />
		<property name="configLocation" value="classpath:mybatis-config.xml"/>
	</bean>
	
	<bean id="sqlSessionTemplate" name="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate" destroy-method="clearCache">
		<constructor-arg index="0" ref="sqlSessionFactoryDefault" />
	</bean>

	<!-- Transaction -->
	<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"/>
	</bean>
	
	<bean id="mapperScan" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="com.sk.*"></property>
		<property name="annotationClass" value="org.springframework.stereotype.Repository"></property>
		<property name="sqlSessionFactoryBeanName" value="sqlSessionFactoryDefault"></property>
	</bean>

</beans>
```

 - web.xml 파일에 mybatis-context.xml 경로를 추가한다. 

```xml
<servlet>
		<servlet-name>integrated-log</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>
                /WEB-INF/context/integratedLogServlet.xml
                /WEB-INF/context/mybatis-context.xml
            </param-value>
		</init-param>
		<load-on-startup>2</load-on-startup>
	</servlet>
```

 -  src/main/resources 하위에 mybatis-config.xml 파일을 생성한다. 

 ```xml
 <?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
	<settings>
		<setting name="callSettersOnNulls" value="true" />			<!-- resultType  Map null 매핑처리 -->
		<setting name="mapUnderscoreToCamelCase" value="true" />	<!-- resultType  Map CamelCase 매핑처리 -->
		<setting name="jdbcTypeForNull" value="VARCHAR" />			<!-- 파라미터에 Null 값이 있을 경우 jdbcType=VARCHAR 로 설정 -->
	</settings>
</configuration>
 ```

![image1](/assets/images/page7/step10.JPG)

 - 톰캣서버의 context.xml 에 DB 커넥션 풀 리소스를 설정한다. 

![image1](/assets/images/page7/step11.JPG)

 - ojdbc6.jar 파일을 클래스 패스에 추가한다. 

 - list.jsp : List<User> 를 받아와, 화면에 바인딩해줌

 ```html
 <%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<link type="text/css" rel="stylesheet" href="/css/default.css" />
</head>
<body>
	<div class="container">
		<div>
			<h2>사용자 목록</h2>
		</div>
		<table class="tables1 o1 mt10">
			<thead>
				<tr>
					<th>id</th>
					<th>이름</th>
					<th>주민번호</th>
					<th>이메일</th>
				</tr>
			</thead>
			<tbody>
				<c:forEach var="user" items="${users}">
					<tr>
						<td class="als">${user.no}</td>
						<td>${user.name}</td>
						<td>${user.jumin}</td>
						<td>${user.email}</td>
					</tr>
				</c:forEach>
			</tbody>
		</table>
	</div>
</body>
</html>
 ```

 - list2.jsp: 클라이언트 사이드 렌더링 방식으로 구현

 ```html
 <%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<link type="text/css" rel="stylesheet" href="/css/default.css" />
</head>
<body>
	<div class="container">
		<div>
			<h2>사용자 목록</h2>
		</div>
		<table class="tables1 o1 mt10">
			<thead>
				<tr>
					<th>id</th>
					<th>이름</th>
					<th>주민번호</th>
					<th>이메일</th>
				</tr>
			</thead>
			<tbody>
				<tr class="template">
					<td class="als"><span class="no" >1</span></td>
					<td><span class="name" >2</span></td>
					<td><span class="jumin" >3</span></td>
					<td><span class="email" >4</span></td>
				</tr>
			</tbody>
		</table>
	</div>
</body>
<script src="/js/jquery-3.4.1.min.js"></script>
<script type="text/javascript">

function send(trid, callbackFunc) {
	
    let _options = {
        dataType: 'json',
        method: 'POST',
        contentType: 'application/json; charset=utf-8'
    };
    
    $.ajax({
        url: trid,
        dataType: _options.dataType,
        method: _options.method,
        contentType: _options.contentType,
        async: true,
        cache: false,
        //data: strData,
        success: function (response) {
        	callbackFunc(response);
        },
        error: function (e) {
            alert(e.responseText);
        }
    });
}

function resultRendering(response){

	const tbody = $('tbody');
	const $tr = $('.template').clone(true);
	
	for(var i=0; i<response.length; i++){
		
		const $tr_i = $tr.clone(true);
		$tr_i.find('.no').text(response[i].no);
		$tr_i.find('.name').text(response[i].name);
		$tr_i.find('.jumin').text(response[i].jumin);
		$tr_i.find('.email').text(response[i].email);
		tbody.append($tr_i);
	} 
}

send('/user/listJson', resultRendering);


</script>
</html>
 ```
 - user.jsp : 1명의 사용자 정보를 보여주는 화면
 - date 타입은 <fmt:formatDate value="${user.date}" pattern="yyyy/MM/dd"/> 로 포맷팅

 ```html
 <%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<link type="text/css" rel="stylesheet" href="/css/default.css" />
</head>
<body>
	<div class="container">
		<div>
			<h2>사용자 상세</h2>
		</div>
		<table class="tables1 o1 mt10">
			<thead>
				<tr>
					<th>id</th>
					<th>이름</th>
					<th>주민번호</th>
					<th>이메일</th>
					<th>등록일</th>
					<th>페이</th>
				</tr>
			</thead>
			<tbody>
				<tr>
					<th>${user.no}</th>
					<th>${user.name}</th>
					<th>${user.jumin}</th>
					<th>${user.email}</th>
					<th><fmt:formatDate value="${user.date}" pattern="yyyy/MM/dd"/>
					 
					</th>
					<th><fmt:formatNumber value="${user.pay}" pattern="###,###,##0"/>원</th>
				</tr>
			</tbody>
		</table>
	</div>
</body>
</html>
 ```

 - put.jsp
 - 데이터 서버 전송을 3가지 방식으로 구현함
 - 첫번째는 form submit : form 에 action 속성을 생략하면, 동일 url 로 post 전송이 된다. 
 - 두번째는 순수 자바스크립트만을 사용하여 ajax 방식으로 전송함
 - onreadystatechange는 XMLHttpRequest 상태가 변할때, 호출하는 함수 지정
 - 상태는 1(open메소드 성공), 2(모든 요청에 대한 응답 도착), 3(요청한 데이터 처리중)등, 4는 요청한 데이터의 처리가 완료되어 응답할 준비가 됨

 - 세번째는 jquery ajax 방식이다. 

 ```html
 <%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<link type="text/css" rel="stylesheet" href="/css/default.css" />
</head>
<body>
	<div class="container">
		<div>
			<h2>사용자 등록 폼</h2>
		</div>
		<h4>사용자 입력</h4>
		<form method="post">
			<div class="boxs mt5 fss14 cls2">
				<label for="name">이름</label> <input class="is2" type="text" name="name" id="name" placeholder="이름을 입력하세요">
			</div>
			<div class="boxs mt5 fss14 cls2">
				<label for="jumin">주민번호</label> <input class="is2" type="text" name="jumin" id="jumin" placeholder="주민번호를 입력하세요">
			</div>
			<div class="boxs mt5 fss14 cls2">
				<label for="email">이메일</label> <input class="is2" type="text" name="email" id="email" placeholder="이메일을 입력하세요">
			</div>
			<div class="boxs mt5 fss14 cls2">
				<label for="pay">페이</label> <input class="is2" type="text" name="pay" id="pay" placeholder="이메일을 입력하세요">
			</div>
			<hr>
			<div>
				<div>
					<button class="btns a" type="submit">등록</button>
					<button class="btns a" onclick="location.href='/user/list'" type="button">취소</button>
				</div>
			</div>
		</form>
		<br>
		<div>
			<button class="btns a" id="submit_ajax_js">등록 Ajax For JS</button>
		</div>
		<br>
		<div>
			<button class="btns a" id="submit_ajax_jquery">등록 Ajax For JQuery</button>
		</div>
		<div>
			<p id="saveName"></p>
			<p id="saveJumin"></p>
			<p id="saveEmail"></p>
		</div>
	</div>
	<!-- /container -->
</body>
<script src="/js/jquery-3.4.1.min.js"></script>
<script>
/* [Ajax javascript]*/
const httpRequest = new XMLHttpRequest();
const submitAjaxJsBtn = document.querySelector('#submit_ajax_js');
submitAjaxJsBtn.addEventListener('click', putRequest);


function putRequest(){

	const name = document.querySelector('#name').value;
	const jumin = document.querySelector('#jumin').value;
	const email = document.querySelector('#email').value;
	const pay = document.querySelector('#pay').value;

	httpRequest.onreadystatechange = alertContents;
	httpRequest.open('POST', '/user/putAjax');
	httpRequest.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded; charset=UTF-8');
	httpRequest.send('name='+name+"&jumin="+jumin+"&email="+email+"&pay="+pay);
	
}

function alertContents() {
	try {
		if (httpRequest.readyState == XMLHttpRequest.DONE) { //4인 경우
			if (httpRequest.status == 200) {
				alert(httpRequest.responseText);
			} else {
				alert('There was a problem with the request.');
			}
		}
	} catch (e) {
		alert('Caught Exception: ' + e.description)
	}
}


// 세번째 방식 : jquery ajax 
const submitAjaxJQueryBtn = $('#submit_ajax_jquery');
submitAjaxJQueryBtn.on('click', function(){

	let param = {
			name : $('#name').val(),
			jumin : $('#jumin').val(),
			email : $('#email').val(),
			pay : $('#pay').val()
	}
	
	send('/user/putAjaxJson', param, resultRendering); /* object to queryString => $.param(param) */
	
});

function resultRendering(response){
	$('#saveName').html(response.name);
	$('#saveJumin').html(response.jumin);
	$('#saveEmail').html(response.email);
}

function send(trid, sendData = {}, callbackFunc) {
	
    let _options = {
        dataType: 'json',
        method: 'POST',
        contentType: 'application/json; charset=utf-8'
    };
    let strData;
    
    if (typeof sendData === 'object') {
        strData = JSON.stringify(sendData);
    }
    else {
        strData = sendData;
    }
    $.ajax({
        url: trid,
        dataType: _options.dataType,
        method: _options.method,
        contentType: _options.contentType,
        async: true,
        cache: false,
        data: strData,
        success: function (response) {
        	callbackFunc(response);
        },
        error: function (e) {
            alert(e.responseText);
        }
    });
}

</script>

</html>
 ```

# Reference
