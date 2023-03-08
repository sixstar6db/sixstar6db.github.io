---
title: "[java] Maven에서 라이브러리 직접 dependency 추가"
excerpt: "Maven에서 라이브러리 직접 dependency 추가"

categories:
  - java-core
tags:
  - [java]

permalink: /java-core/maven_local_libarary_dependency/

toc: true
toc_sticky: true

date: 2023-03-07
last_modified_at: 2023-03-07
---

## Maven에서 라이브러리 직접 dependency 추가

#### 간혹 라이브러리 의존성 관리할때 maven repository 에서 다운로드 받지 않고, 로컬pc에 있는 jar 파일을 직접 설정해줘야 하는 경우가 있다.

> lib 폴더내에 jar 파일 이동
 - 소스코드 프로젝트 root 경로에 lib 폴더를 만든 후 jar 파일을 넣어둔다.

> pom.xml 설정

```xml
<dependency>
    <groupId>org.dblife.custom</groupId>
    <artifactId>library-producer</artifactId>
    <version>1.0</version>
    <scope>system</scope>
    <systemPath>${basedir}/lib/library-producer-1.0.jar</systemPath>
</dependency>
```

 - ${basedir}은 프로젝트 root 경로이다.
 - scope 을 system 으로 해준다. 

> build 해보기
 - 프로젝트를 빌드 하면, 해당 jar 파일이 누락된 상태로 실행jar가 만들어진다. 
 - 아래와 같이 springboot plugin 설정에 includeSystemScope을 true 로 해준다.다시 빌드 하면 해당 jar가 포함되어 들어간다.

```xml
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <includeSystemScope>true</includeSystemScope>
        </configuration>
    </plugin>
</plugins>
```
