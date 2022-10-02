---
title: "[docker] 도커-Mariadb 설치해보기"
excerpt: "도커 사용법"

categories:
  - tools
tags:
  - [docker]

permalink: /tools/docker_mariadb/

toc: true
toc_sticky: true

date: 2022-06-07
last_modified_at: 2022-06-07
---

## 윈도우 도커 설치

 - 유튜브에 보면 친절하게 설치 가이드를 제공해주는 채널이 있다.
 - 추천 : https://youtu.be/Fs7JoQYEB4U

 - 별다른 설정없이 윈도우용 설치 파일을 받은 후 설치를 진행하면 된다.
 - 중간에 한번 리눅스커널 업데이트를 해주어야 한다.  

## 🦥 도커에 mariadb

>  컨테이너 삭제

  `docker rm mariadb`

> mariadb 이미지 다운로드

  `docker pull mariadb`  

> 컨테이너 생성 및 실행

 - --name 컨테이너 이름 정의
 - -d 컨테이너를 백그라운드에서 실행
 - -p 호스트와 컨테이너간의 포트를 연결(host-port:container-port)
 - 

  `docker run -p 3306:3306 --name mariadb -e MYSQL_ROOT_PASSWORD=root -d mariadb`

> mysql 접속

  `docker exec -it mariadb /bin/bash`

> mariadb 로그인

  `mysql -u root -p`

> mariadb 초기 설정

 - 데이터베이스 보기 `show databases;`
 - 데이터베이스 생성 `create database mydb;`
 - 데이터베이스 변경 `use mydb`
 - 사용자 생성 `CREATE USER 'user'@'%' IDENTIFIED BY 'user';`
 - 권한 부여 `GRANT ALL PRIVILEGES ON *.* TO 'user'@'%';`

 - 권한 flush `FLUSH PRIVILEGES;`

> 컨테이너 중지

  `docker stop mariadb`

# Reference

> - 
