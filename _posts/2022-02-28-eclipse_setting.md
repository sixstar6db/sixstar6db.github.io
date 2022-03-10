---
title:  "[IDE] 이클립스 최초 설정 몇가지"
excerpt: "이클립스 설정"

categories: Java
tags:
  - [IDE]

toc: true
toc_sticky: true
 
date: 2022-02-28
last_modified_at: 2022-02-28
author_profile: false     
---

## 이클립스 최초 셋팅

**[1] SpringToolSuite4.ini (STS.ini, eclipse.ini)**
```xml
-vm
C:/Program Files/Java/jre1.8.0_131/bin/javaw.exe
-vmargs
-Dosgi.requiredJavaVersion=1.8
-Xms1024m
-Xmx2048m

```
-vm 옵션: eclipse 가 사용할 jdk ( –vmargs  보다 먼저 명시)

**[2]Preferences 설정**

1. 먼저 다크 모드로 변경

![image1](/images/page6/darkmode.JPG)

2. UTF-8 설정

![image1](/images/page6/utf8-1.JPG)

 - text content-type 도 utf-8 로 변경

![image1](./images/page6/utf8-2.JPG)

 - jsp, html, css 파일인코딩 UTF-8로 설정

![image1](./images/page6/utf8-3.JPG)
![image1](./images/page6/utf8-4.JPG)
![image1](./images/page6/utf8-5.JPG) 

- 스펠링 체크 해제(약간의 IDE 속도 성능 개선)

![image1](./images/page6/spelling.JPG) 

- 폰트 설정

![image1](./images/page6/font.JPG) 

- startup and shutdown ( 시작할때 실행되는 플러그인 체크 해제하여 속도 개선)

![image1](./images/page6/startup.JPG) 

- 망분리 환경에서 개발 할 경우 아래 maven 체크

![image1](./images/page6/maven.JPG) 

- validation 체크를 하지 않도록 설정한다. 

![image1](./images/page6/validation.JPG) 

# Reference
