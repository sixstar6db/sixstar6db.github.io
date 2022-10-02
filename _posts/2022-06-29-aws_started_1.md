---
title: "[docker] AWS에 EC2와 RDS 생성하기"
excerpt: "AWS에 EC2와 RDS 생성하기"

categories:
  - tools
tags:
  - [aws]

permalink: /tools/aws_started_1/

toc: true
toc_sticky: true

date: 2022-06-29
last_modified_at: 2022-06-29
---

## 🦥 AWS 가입하기

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

![image1](/assets/images/page10/img5.png)
![image1](/assets/images/page10/img6.png)

> 네트워크 설정

- 보안그룹은 중요한 설정이다. SSH트래픽 허용에 `내IP` 를 선택한다. 지정된 IP에서만 SSH접속이 가능하도록 하는 것이 안전하다.  

![image1](/assets/images/page10/img7.png)

> 스토리지 설정

- 30GB로 설정한다. 

![image1](/assets/images/page10/img8.png)

> 인스턴스 시작

![image1](/assets/images/page10/img9.png)

> Xshell 을 이용해 서버 터미널 접속

- 인스턴스를 시작할때마다, 퍼블릭IP4v 주소에 IP가 변경된다. 
- Xshell 등록 정보를 매번 변경해주어야 한다. 탄력적IP를 사용해 고정 IP를 사용할 수 있으나, 비용이 발생한다. 

![image1](/assets/images/page10/img10.png)
![image1](/assets/images/page10/img15.png)

> pem 키 등록

 - Xshell 에 pem 키를 등록한다. 사용자 인증 메뉴이 들어가, 사용자 이름을 입력한다. 사용자 이름은 `ec2-user`이다. 
 - 아래 그림과 같이 진행한다. 

![image1](/assets/images/page10/img11.png)
![image1](/assets/images/page10/img12.png)
![image1](/assets/images/page10/img13.png)
![image1](/assets/images/page10/img14.png)

 - 연결 버튼을 누른다. 

## 기본 설정

> 자바 설치

 `sudo yum install -y java-1.8.0-openjdk-devel.x86_64`

> 타임존 변경

- `date` 명령어를 입력해보면, 미국 시간에 맞춰줘 있는 것이 보인다. 

- 다음 명령어를 수행한다. 
  1. `sudo rm /etc/localtime`
  2. `sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime`

## RDS 생성하기

- rds 로 검색하고 클릭한다. 데이터베이스 생성 버튼을 클릭한다. 

![image1](/assets/images/page10/img16.png)
![image1](/assets/images/page10/img17.png)

 - Maria DB 를 선택한다. 

![image1](/assets/images/page10/img18.png)

- 프리티어를 선택하고, 설정정보를 입력한다. 
![image1](/assets/images/page10/img19.png)
![image1](/assets/images/page10/img20.png)
![image1](/assets/images/page10/img21.png)
![image1](/assets/images/page10/img22.png)

- 퍼블릭 엑세스를 Y로 선택한다. 

![image1](/assets/images/page10/img23.png)

- 데이터 베이스 생성버튼을 클릭한다. 약 5분 이상 소요된다. 
![image1](/assets/images/page10/img24.png)


- RDS의 몇 가지 설정 진행을 위해, 파라미터 그룹을 선택한다. 
![image1](/assets/images/page10/img25.png)

- 파라미터 그룹 생성을 클릭한다. 
![image1](/assets/images/page10/img26.png)

- 설치된 MariaDB 와 동일한 버전을 선택하고, DB 파라미터 그룹 이름을 입력한다. 난 DB 인스턴스 식별자와 동일하게 넣어줬다. 
![image1](/assets/images/page10/img27.png)

- 생성한 파라미터를 클릭하고, 파라미터 편집을 클릭한다. 
![image1](/assets/images/page10/img28.png)
![image1](/assets/images/page10/img29.png)

- 먼저 파라미터 검색을 통해 `time_zone`을 찾는다. 아래와 같이 `Asia/seoul`을 선택한다.
![image1](/assets/images/page10/img30.png)

- 파라미터 검색에 `character_set`을 찾아서 `utf8mb4` 를 선택한다. utf8mb4 는 이모지가 가능하다고 한다. 
- `collation` 을 검색해서 나오는 것은 `utf8mb4_generel_ci` 를 선택하였다. (utf8mb4 항목이 없어, 뭔가 일반적인 느낌이 나는 걸로 선택했다.)
![image1](/assets/images/page10/img31.png)
![image1](/assets/images/page10/img32.png)

- `max_connection` 은 150으로 설정한다.
![image1](/assets/images/page10/img33.png)

- 파라미터 그룹을 수정하기 위해 아래와 같이 한다. 
![image1](/assets/images/page10/img34.png)
![image1](/assets/images/page10/img35.png)

- DB 파라미터 그룹에 새로 만든 파라미터 그룹을 선택한다. 
![image1](/assets/images/page10/img36.png)

- 수정사항을 즉시적용으로 선택하고, 수정한다. 만약 운영중인 서비스라면, 새벽시간대로반영할 수 있게 예약반영을 선택하면 된다.
![image1](/assets/images/page10/img37.png)

- 보안그룹의 인바운드 규칙을 설정한다. 
- 내 PC와 EC2 서버에서 접근할 수 있또록 인바운드 규칙을 2개 설정한다. 
- 소스에 EC2 서버의 보안그룹 ID를 입력하면 된다. 

![image1](/assets/images/page10/img38.png)
![image1](/assets/images/page10/img39.png)

- HeidiSQL 을 통해 DB에 접속해보자. 
- 연결/보안에서 엔드포인트를 복사 한후, HeidiSQL 접속 정보에 입력한다. 
- 처음 인스턴스 구성시 생성한 database 명과 id, pwd 를 입력한 후 연결한다. 

![image1](/assets/images/page10/img40.png)
![image1](/assets/images/page10/img41.png)

- AWS에서 파라미터 그룹 설정이 제대로 적용되었는지 확인해본다. 일부 몇개는 적용이 제대로 안되었다. 
- 직접 수정한다!
![image1](/assets/images/page10/img42.png)

- EC2 에 접속하여 DB 접속을 해본다. 

`sodu yum install mysql`

`mysql -u 계정 -p -h Host 주소`

## 과금

 - AWS 사용중 얼마 되지 않아, 과금이 된 것을 확인하였다. 

![image1](/assets/images/page10/img45.png) 

 - 구글링 해보니, EC2 인스턴스를 2개 이상 만들어서 그랬던 것. 인스턴스 별로 기본 스토리지가 할당이 되는데, 사용을 하든 안하든 비용이 청구되는 방식인것으로 보인다.

# Reference

> - 
