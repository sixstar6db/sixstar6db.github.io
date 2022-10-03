---
title: "[java] 자바 버전 업그레이드 이슈"
excerpt: "자바 버전 업그레이드 이슈"

categories:
  - java-advanced
tags:
  - [java, spring]

permalink: /java-advanced/jpa_learned/

toc: true
toc_sticky: true

date: 2022-09-24
last_modified_at: 2022-09-24
---

## java8 업그레이드 이슈

 - jvm 런트임 환경은 java8 로 설정이 되어 있고, java 소스코드는 jdk7 로 컴파일해서 배포하는 스프링프레임워크 기반의 어플리케이션이 있었다. 
 굳이 jdk7을 쓸필요가 없을것 같아, class 파일 하나만 8버전으로 컴파일해서 배포했는데, 심각한 오류가 발생하였다. <br>

 `ASM ClassReader failed to parse class file - probably due to a new Java class file version that isn't supported yet: file  `

  - 구글링을 해보았더니, 스프링프레임워크 4.3.x 버전부터 java8 을 지원하였다. 그런데 해당 어플리케이션은 3 버전을 사용하고 있었다. 스프링 버전을 올리는 것은 쉽지 않아, 일단 자바 버전 올리는 것은 보류하기로 하였다.ㅜ

 - 22년도 4분기에 나올 스프링 6 버전, 스프링부트3 은 최소 java17이 필요하다고 한다. 

## java 버전 정리

 - LTS(Long Term Support) : java8 / java11 / java17

 - java8 : java11 이나 java17 에 비해 EOL(End Of Life)가 긴 제품이 많다 보니, 제일 많이 쓰이고 있는 버전이다. 구글링 해보니, 오라클은 2030년이라고 한다. 스프링6 버전이 java 17 이상이 최소 요건이라고 하면, 앞으로 java8 다음으로 많이 사용하게 될 자바 버전은 java17이 될 것으로 보인다.  

# Reference

 - 
