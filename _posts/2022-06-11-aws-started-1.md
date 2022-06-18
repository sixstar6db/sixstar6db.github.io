---
title:  "[기타] AWS 시작하기"
excerpt: "AWS 시작하기"

categories: Java
tags:
  - [other]

toc: true
toc_sticky: true
 
date: 2022-06-11
last_modified_at: 2022-06-11
author_profile: false     
---

## AWS 가입하기

  - AWS는 이메일 단위로 가입할 수 있다. 예전에 가입했다가 계정중지가 되었는데, 그 메일 계정은 더이상 사용할수 없었다. 
  - AWS가입시 해외결제가 가능한 신용카드가 필요하다. 가입 방법은 구글링 통해 쉽게 학인 가능하여 생략~

## AWS EC2 프리페어 사용해보기

![image1](/assets/images/page10/img1.PNG)

위 화면에서 EC2를 클릭한다. 

![image1](/assets/images/page10/img2.PNG)

 - 리소스 현황을 보면 인스턴스(실행중)이 0개이고, 인스턴스가 1개로 되어 있다. 이 상태에서 인스턴스 1개를 더 만들어보자. `리소스` 아래에 `인스턴스 시작` 메뉴가 있다. 주황색 버튼으로 된 버튼을 클릭한다.

 - 인스턴스 이름을 입력한다. 

![image1](/assets/images/page10/img3.PNG)

> OS 이미지를 선택한다. Amazon Linux 를 선택한다. 

- 아마존 리눅스를 선택하는 이유:
  1. 아마존에서 개발하고 있기 때문에 지원받기 쉽다.
  2. 레드햇 베이스라 익숙하다. 
  3. AWS 각종 서비스와 호환이 좋다.
  4. Amazon 독자적인 개발 레파지토리를 사용하고 있어 yum이 빠르다. 

![image1](/assets/images/page10/img4.PNG)  

> 키 패어 생성

 - 인스턴스로 접근하기 위해서는 pem키(비밀키)가 필요하다. 인스턴스는 pem키와 매칭되는 공개키를 가지고 있어, 해당 pem키 외에는 접근을 허용하지 않는다. pem 키는 분실되지 않도록 관리를 철저히 해야 한다. 

![image1](/assets/images/page10/img5.PNG)
![image1](/assets/images/page10/img6.PNG)

> 네트워크 설정

- 보안그룹은 중요한 설정이다. SSH트래픽 허용에 `내IP` 를 선택한다. 지정된 IP에서만 SSH접속이 가능하도록 하는 것이 안전하다.  

![image1](/assets/images/page10/img7.PNG)

> 스토리지 설정

- 30GB로 설정한다. 

![image1](/assets/images/page10/img8.PNG)

> 인스턴스 시작

![image1](/assets/images/page10/img9.PNG)

> Xshell 을 이용해 서버 터미널 접속

- 인스턴스를 시작할때마다, 퍼블릭IP4v 주소에 IP가 변경된다. 
- Xshell 등록 정보를 매번 변경해주어야 한다. 탄력적IP를 사용해 고정 IP를 사용할 수 있으나, 비용이 발생한다. 

![image1](/assets/images/page10/img10.PNG)
![image1](/assets/images/page10/img15.PNG)

> pem 키 등록

 - Xshell 에 pem 키를 등록한다. 사용자 인증 메뉴이 들어가, 사용자 이름을 입력한다. 사용자 이름은 `ec2-user`이다. 
 - 아래 그림과 같이 진행한다. 

![image1](/assets/images/page10/img11.PNG)
![image1](/assets/images/page10/img12.PNG)
![image1](/assets/images/page10/img13.PNG)

 - 연결 버튼을 누른다. 

## 기본 설정

> 자바 설치

 `sudo yum install -y java-1.8.0-openjdk-devel.x86_64`

> 타임존 변경

- `date` 명령어를 입력해보면, 미국 시간에 맞춰줘 있는 것이 보인다. 

- 다음 명령어를 수행한다. 
  1. `sudo rm /etc/localtime`
  2. `sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime`

# Reference

> - 
