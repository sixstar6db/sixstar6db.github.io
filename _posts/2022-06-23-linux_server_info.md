---
title: "[linux] 리눅스 서버 정보 확인하기"
excerpt: "리눅스 서버 정보 확인하기"

categories:
  - server
tags:
  - [linux]

permalink: /server/linux_server_info/

toc: true
toc_sticky: true

date: 2022-06-23
last_modified_at: 2022-06-23
---

## CPU 정보

> 논리 프로세서란?

- 프로세서는 CPU와 동일하다고 보면 된다. 일반적으로 1코어당 1쓰레드라고 하는데, 하이퍼쓰레딩이라는 기술때문에 1코어당 2쓰레드를 만들 수 있다고 한다. 그럼 8개의 논리 프로세서가 있다고 할 수 있다.

- 리눅스에서 `lscpu` 명령어를 입력하면 아래와 같은 정보가 출력된다. 
  1. CPU(s) : 6  -> 6개의 프로세서 
  2. Thread per core : 1 <br>
즉, 6x1 = 6 개의 논리 프로세서 가 존재, 6개의 논리코어/6개의 물리코어가 존재한다고 얘기할 수 있다.

![image1](/assets/images/page15/img1.JPG)

 - `cat /proc/cpuinfo` 명령어를 통해서도 cpu 정보를 확인할 수 있다. 
 - 필요한 정보만 추출하여 보고 싶다면, <br>

![image1](/assets/images/page15/img2.JPG)

## 메모리 정보

> top

 - top 명령어는 현재 OS의 상태를 나타내준다. 
 - Load Average : CPU Load 의 1분, 5분, 15분에 대한 평균값을 나타낸다. 싱글코어일 경우 1.0의 값은 CPU를 100% 사용하고 있다는 의미이다. 
 - KiB Mem : 49294116 KiB 이란 값은 대략 50GB를 의미한다. 
 - KiB Swap : Mem 은 물리적 메모리 공간, Swap 은 스왑메모리라고 불리우는 디스크 공간을 의미한다. 아래 화면에서는 약 5GB의 스왑메모리가 잡혀있다.

![image1](/assets/images/page15/img3.JPG)

 - `free -m` 명령어를 수행하면, 아래와 같은 결과가 나온다. 
 - used : 현재 사용중인 메모리 크기이다. (total - free - buffer/cache)
 - free : 사용되지 않은 메모리 양으로 커널 또는 애플리케이션에서 사용 가능하다. 
 - buffer / cache : 커널이 성능 향상을 위해 캐시 영역으로 사용하는 메모리 크기
 - available : swapping 없이 새로운 애플리케이션을 실행 가능한 가용 메모리 크기
![image1](/assets/images/page15/img4.JPG)

 - `cat /proc/meminfo` 명령어를 수행하면 아래와 같은 결과가 나온다.

![image1](/assets/images/page15/img5.JPG)

 - Active : 가장 최근에 사용된 메모리 영역
 - Inactive : 참조된지 좀 오래된 메모리 영역

- OS는 자원이 유휴상태에 있는 것을 좋아하지 않으므로, 남는 메모리는 버커/캐시 영역으로 활용한다. 캐시 용도로 사용되다가 프로세스에서 메모리를 추가적으로 요청할때, 캐시메모리를 반환 한 후 요청한 프로세스 메모리로 할당한다. 또는 기존 프로세스 Inactive 메모리를 SWAP 영역으로 내려쓴 후 요청한 프로세스 메모리로 할당한다. Incative 메모리를 반환하지 않는 이유는 메모리에 어떤 내용이 있는지 알 수 없기 때문이다. 

# Reference