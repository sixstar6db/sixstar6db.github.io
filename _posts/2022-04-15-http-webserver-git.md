---
title:  "[java] http 웹서버 만들기 1 - git 연동"
excerpt: "http 웹서버 만들기 1 - git 연동"

categories: Java
tags:
  - [java]

toc: true
toc_sticky: true
 
date: 2022-04-15
last_modified_at: 2022-04-15
author_profile: false     
---

## github 레파지토리에서 이클립스로 import 하는 과정

![image1](/assets/images/page9/img1.PNG)

 - open Perspective 에서 git을 클릭한다. 

![image1](/assets/images/page9/img2.PNG)

 - github 에 가서 git 레파지토리 url 을 copy 한다. 

 - 이클립스에서 Clone a Git repository 를 클릭한다.

 ![image1](/assets/images/page9/img3.PNG)

 - copy 한 git 정보가 화면에 나온다. Authentication 은 처음에 빈 칸으로 나오는데, 입력 후 Next 를 클릭한다. 

![image1](/assets/images/page9/img4.PNG)

 - 브랜치 정보가 나온다. 전체 선택하고, Next 를 클릭한다. 

![image1](/assets/images/page9/img5.PNG)

 - 로컬 스토리지를 지정한다. 최초에는 user 경로에 git 하위로 지정된다. 
 - C:\spring\git 로 변경하고 finish 를 클릭한다. 

![image1](/assets/images/page9/img6.PNG)

 - 위에 처럼 나오는데, Working Tree 에서 우클릭한 후 Import project 를 클릭한다. finish 버튼을 눌러 완료 한다. 

 ![image1](/assets/images/page9/img7.PNG)

  - Java EE  버튼을 클릭하면 프로젝트가 나온다. 
  - 만약 Maven 프로젝트가 아니라면, congiure 에서 convert maven project 를 해줌.

 ![image1](/assets/images/page9/img8.PNG)

  - WebServer.java 에서 main 을 실행한 후, 웹 브라우저에서 접속해본다. 

# Reference

> - [백명석의 클린 코더스](https://)
