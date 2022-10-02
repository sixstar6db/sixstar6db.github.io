---
title: "[java] 스프링부트 웹서비스 AWS 구현(2)-개발"
excerpt: "스프링부트 웹서비스 AWS 구현(2)-개발"

categories:
  - service-develop
tags:
  - [spring, aws]

permalink: /service-develop/aws_springboot_web2/

toc: true
toc_sticky: true

date: 2022-08-28
last_modified_at: 2022-08-28
---

## 🦥 EC2서버에 배포하기

> AWS에서 EC2 와 RDS를 생성한다.(이전 블로그 참고)

> EC2에 프로젝트 Clone 

 - `sudo yum install git`
 - `git --version`

 - 프로젝트코드 보관할 디렉토리 생성
 - `mkdir app`
 - `mkdir ~/app/step1`

 - github에 가서 프로젝트코드 git https 주소를 복사한다.
 - `git clone 복사한주소` 

![image1](/assets/images/page11/img23.png) 

 - 테스트를 진행한다. 처음엔 의존성 라이브러리등을 다운로드 하느라 오래 걸린다. 
 - 테스트 실패시, 소스 코드 수정 후 다시 git 에 커밋하고 `git pull`을 수행한다.

![image1](/assets/images/page11/img24.png)

- http 요청에 대한 8080 포트 열기 
![image1](/assets/images/page11/img25.png)

> 배포 쉘 스크립트 작성하기

 - `vim ~/app/step1/deploy.sh`
 
 ![image1](/assets/images/page11/img26.png)
 ![image1](/assets/images/page11/img27.png)

 - `git pull` : 마스터 브랜치의 최신 내용을 받는다. 
 - `JAR_NAME=$(ls -tr $REPOSITORY/ | grep jar | tail -n 1)` : tail -n 으로 가장 나중에 생긴 jar 파일을 찾는다. 
 - `nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &` 백그라운드로 jar 파일을 실행시킨다. nohup.out 에 로그가 출력된다. 

> RDS 연동하기 

 - Posts 테이블 생성한다. 테스트를 돌려본 후 로그에 출력되는 테이블 생성 스크립트를 복사한다. 

```sql 
create table posts 
(id bigint not null AUTO_INCREMENT
, create_date DATETIME
, modified_date DATETIME
, author varchar(255)
, content TEXT not NULL
, title varchar(500) not NULL
, primary key (id)) 
engine=INNODB
```

- heidisql 에 접속한 후 테이블을 생성한다. 
- Mariadb JDBC 드라이버를 build.gradle 에 추가한다. 
 `compile("org.mariadb.jdbc:mariadb-java-client")`
-application-real.properties 파일을 생성한다. profile=real 환경에서 적용된다.  
-운영에 적용될 설정 정보를 작성한다. 

```java
spring.profiles.include=real-db
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
```

-EC2 서버에 DB 설정 파일을 생성한다.
-vim ~/app/application-real-db.properties

```java
spring.jpa.hibernate.ddl-auto=none
spring.datasource.url=jdbc:mariadb://rds주소:포트/database명
spring.datasource.username=db계정
spring.datasource.password=db계정비번
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
```

- deploy.sh가 real profile 을 사용할 수 있도록 변경한다. 

```java
nohup java -jar \
 -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-real-db.properties,classpath:/application-real.properties \
  -Dspring.profiles.active=real \
   $REPOSITORY/$JAR_NAME 2>&1 &
```

 - -Dspring.profiles.active=real : application-real.properties 를 활성화 시킨다. spring.profiles.include=real-db 옵션때문에 real-db 도 같이 활성화된다. 

 - 위에 java 실행 명령어는 -D 파라미터를 사용하였는데, 아래와 같이 사용해도 정상적으로 실행됨.

```java
// 첫번째 방법
java -jar springboot-ws-started-1.0-SNAPSHOT.jar --spring.config.location=/home/ec2-user/app/application-real-db.properties --spring.profiles.active=real

// 두번째 방법
java -jar springboot-ws-started-1.0-SNAPSHOT.jar --spring.config.name=application-real-db --spring.config.location=/home/ec2-user/app/ --spring.profiles.active=real
```

 - 각 옵션의 속성에 대한 값이 여러 개일 경우 ','로 구분한다. 
 ex ) `--spring.config.name=application,jdbc`

 - 만약 경로를 잘못 넣어서 maria db 설정 파일을 찾지 못한다면? maria db 대신 h2 가 적용된다. 
 - 위 옵션은 프로그래밍에 적용할수도 있다. (PropertySourcesPlaceholderConfigurer )


## Travis CI 사용

> CI/CD  란 ?

 - CI는 코드버전관리 시스템에 PUSH 되면, 자동으로 테스트와 빌드가 수행되어 안정적인 배포 파일을 만드는 과정을 말함.
 - CD는 이 빌드 결과를 운영서버에 무중단 배포까지 진행되는 과정을 말함.

> https://app.travis-ci.com/

 - Travis 는 깃허브에서 제공하는 무료 CI 서비스이다. 
 - Travis 홈페이지에 접속하여, 간단한 설정

```yml
language: java
jdk:
  - openjdk8

#마스터 브랜치만 CI
branches:
  only:
    - master

# cache 기능을 통해 같은 의존성은 다음 배포때부터 다시 받지 않도록 설정
# Travis CI 서버의 home
cache:
  directories:
    - '$HOME/.m2/repository'
    - '$HOME/.gradle'

# 마스터 브랜치에 푸쉬되었을 때 수행하는 명령어
script: "./gradlew clean build"

# CI 실행 완료시 메일로 알람
notifications:
  email:
    recipients:
      - sixstar6@naver.com
``` 

 - 아래와 같은 오류 발생시, 접근권한을 푸는 스크립트를 추가한다. 

```yml
before_install:
  - chmod +x gradlew
``` 

![image1](/assets/images/page11/img28.png)

 - 빌드에 성공함

![image1](/assets/images/page11/img29.png)

## AWS S3 연동하기

> AWS Key 발급

 : AWS서비스에 외부 서비스가 접근할수 있도록 Key 생성, IAM(Identity and Access Management)를 통해 Travis CI가 AWS의 S3와 CodeDeploy에 접근할수 있도록 한다. 

![image1](/assets/images/page11/img30.png)

![image1](/assets/images/page11/img31.png)

![image1](/assets/images/page11/img32.png)

![image1](/assets/images/page11/img33.png)

![image1](/assets/images/page11/img34.png)

![image1](/assets/images/page11/img35.png)

- 최종생성 완료 되면 엑세스 키와 비밀 엑세스키가 생성된다.

![image1](/assets/images/page11/img36.png)

<!--
 - github 에서 AWS에서 생성한 키를 등록한다. 

![image1](/assets/images/page11/img37.png)

![image1](/assets/images/page11/img38.png) 

-->

- Activate 진행이 필요(?)
![image1](/assets/images/page11/img39.png)

- github 로 이동하여, 레파지토리 선택 
![image1](/assets/images/page11/img40.png)

-  more option > settings 를 선택 한후, AWS에서 생성한 키를 등록한다.  

![image1](/assets/images/page11/img41.png)
![image1](/assets/images/page11/img42.png)

## S3 버킷 생성

 - S3 는 파일서버의 일종이다. 게시판서비스라면, 첨부파일 등록을 구현할 때 많이 사용한다.
 - 버킷이란 S3에서 생성할 수 있는 최상위 디렉토리의 개념으로 버킷을 먼저 생성한다.

![image1](/assets/images/page11/img43.png)

 - 버킷명을 작성한다.

![image1](/assets/images/page11/img44.png)

 - 보안을 위해 퍼블릭 엑세스는 차단한다. IAM 사용자로 발급받은 키를 사용해서만 접근 가능하다. 

![image1](/assets/images/page11/img45.png)

- 버킷 생성이 완료 되었다. 

![image1](/assets/images/page11/img46.png)

- .travis.yml 파일을 수정한다.

```yml
before_deploy: # deploy 명령어가 실행되기 전에 수행한다. 
  - zip -r springboot-ws-started * # CodeDeploy 는 jar파일을 인식하지 못하므로 zip 으로 압축, 명령어 마지막은 본인의 프로젝트명.
  - mkdir -p deploy # deploy 라는 디렉토리를 생성한다. Travis CI가 실행중인 위치에서 생성한다. 
  - mv springboot-ws-started.zip deploy/springboot-ws-started.zip # 압축파일을 deploy 하위로 이동
```

 - deploy 명령어가 실행되기 전에 수행한다. 

 ```yml
deploy: # S3로 파일 업로드 혹은 CodeDeploy로 배포등 외부서비스와 연동될 행위 선언
  -provider: s3 
  access_key_id: $AWS_ACCESS_KEY # Travis repo settings 에 설정된 값
  secret_access_key: $ACCESS_SECRET_KEY # Travis repo settings 에 설정된 값
  bucket: sixstar6-aws-build # s3 버킷
  region: ap-northeast-2
  skip_cleanup: true
  acl: private # zip 파일 접근을 private 으로
  local_dir: deploy # before_deploy 에서 생성한 디렉토리
  wait-until-deployed: true
 ```

 - travis 에서 제공하는 provider 가 여러개 있다. 
 - travis 에 설정된 키 이름이 AWS_ACCESS_KEY가 아니라 오류남, 아래 값으로 수정

```yml
  access_key_id: $AWS_ACCESS_KEY_ID # Travis repo settings 에 설정된 값
  secret_access_key: $AWS_SECRET_ACCESS_KEY # Travis repo settings 에 설정된 값
``` 

- 빌드 완료함.

![image1](/assets/images/page11/img47.png)

- AWS S3 에서 확인해보면, 파일이 올라와 있음.

![image1](/assets/images/page11/img48.png)

## Travis CI 와 AWS S3, CodeDeploy 연동하기

 - AWS의 배포시스템인 CodeDeploy 를 이용하기 전에 배포대상인 EC2가 CodeDeploy를 연동 받을 수 있게 IAM 역할을 하나 생성한다. 

 - AWS에서 IAM을 검색하고, 역할을 만든다. 

![image1](/assets/images/page11/img50.png)

 - 잠깐, IAM의 사용자와 역할은 어떤 차이가 있을까?
   : 역할은 AWS 서비스에만 할당할 수 있는 권한, EC2, CodeDeploy등
   : 사용자는 AWS 서비스외에 사용할 수 있는 권한. 로컬PC, IDC서버등 

 - EC2에서 사용할 것이므로 역할 생성

![image1](/assets/images/page11/img51.png)

 - EC2RoleforAWS-CodeDeploy 선택 

![image1](/assets/images/page11/img52.png)

![image1](/assets/images/page11/img53.png)

- 역할이 생성됨.

![image1](/assets/images/page11/img54.png)

 - EC2로 이동하여, 인스턴스 목록으로 이동한뒤, 우클릭하고, 보안 > IAM 역할 수정을 선택한다. 
![image1](/assets/images/page11/img55.png)

 - 방금 생성한 역할을 선택
![image1](/assets/images/page11/img56.png)

 - EC2 인스턴스를 재부팅 한다. 

 - EC2에 접속하여 다음 명령어를 입력한다. 내려받기가 성공하면, 

`aws s3 cp s3://aws-codedeploy-ap-northeast-2/latest/install . --region ap-northeast-2`

`chmod +x ./install`

`sudo ./install auto`

 - 갑자기 "/usr/bin/env: ruby: No such file or directory" 이런 오류가 발생한다.

 `sudo yum install ruby` 으로 ruby 설치.

 `sudo ./install auto` 다시 명령어 실행

`sudo service codedeploy-agent status`

 - "The AWS CodeDeploy agent is running as PID xxxx" 메시지 출력시 정상

 - CodeDeploy 에서 EC2 에 접근하려면 마찬가지로 권한이 필요하고, AWS서비스이니 IAM 역할을 생성한다. 

![image1](/assets/images/page11/img57.png)

![image1](/assets/images/page11/img58.png)

> Code Build 와 CodeDeploy

 - Code Build : Travis CI 와 같은 빌드용 서비스, 규모가 있는 서비스에서는 대부분 젠킨스/팀시티등을 이용하여 거의 사용할 일이 없다. 
 - CodeDeploy : AWS의 배포서비스로서, 오토 스케일링 그룹 배포, 블루 그린 배포, 롤링 배포, EC2 단독 배포등 많은 기능을 지원한다. 
 CodeBuild 는 대체제가 많지만, CodeDeploy 는 대체제가 없다고 함.

> CodeDeploy 어플리케이션 생성

![image1](/assets/images/page11/img59.png)

![image1](/assets/images/page11/img60.png)

- 배포 그룹을 생성한다. 
- 본인이 배포할 서비스가 2대 이상이라면 블루/그린을 선택한다. 난 1대의 EC2에만 배포하므로, 현재위치 선택

![image1](/assets/images/page11/img61.png)

![image1](/assets/images/page11/img62.png)

- 로드밸런싱은 해제한다. 

![image1](/assets/images/page11/img63.png)

## Travis CI, S3, CodeDeploy 연동

 - S3에서 넘겨줄 Zip 파일을 저장할 디렉토리 생성
  1. /app/step2
  2. /app/step2/zip , zip파일 복사 경로

 - Travis CI 설정은 .travis.yml 로 진행, AWS CodeDeploy의 설정은 appspec.yml 로 진행함. 

```yml
version: 0.0 # CodeDeploy 버전, 프로젝트 버전이 아니므로 0.0외의 다른 버전은 오류 발생
os: linux
files:
  - source:  / #CodeDeploy에서 전달해준 파일중 destination으로 이동시킬 대상 지정
    destination: /home/ec2-user/app/step2/zip
    overwrite : yes #파일들을 덮어쓰게 됨
```
 - .travis.yml 에 추가

```yml
  -provider: codedeploy
  access_key_id: $AWS_ACCESS_KEY_ID # Travis repo settings 에 설정된 값
  secret_access_key: $AWS_SECRET_ACCESS_KEY # Travis repo settings 에 설정된 값
  bucket: sixstar6-aws-build # s3 버킷
  key: springboot-ws-started.zip # S3 버킷에 저장된 springboot-webservice.zip 파일을 EC2로 배포
  bundle_type: zip
  application: springboot-ws-started # 웹 콘솔에서 등록한 CodeDeploy 애플리케이션
  deployment_group: springboot-ws-started-group
  region: ap-northeast-2
  wait-until-deployed: true
``` 

 - `Version 2 of the Ruby SDK will enter maintenance mode as of November 20, 2020. To continue receiving service updates and new features, please upgrade to Version 3. More information can be found` 라고 trivis에서 오류가 발생했다. AWS에서도 오류 내역이 보인다. 

![image1](/assets/images/page11/img64.png)

`The deployment failed because no instances were found for your deployment group. Check your deployment group settings to make sure the tags for your Amazon EC2 instances or Auto Scaling groups correctly identify the instances you want to deploy to, and then try again.`

 - yml 파일 작성시, 복사-붙여넣기를 해서 그런게 아닐까 싶다.

- CodeDeploy 로그 경로 

/opt/codedeploy-agent/deployment-root/...
/var/log/aws/codedeploy-agent 
/opt/codedeploy-agent/deployment-root/deployment-logs 에서 로그 확인

 - 이상하게도 S3 에 파일 업로드 하고 codedeploy 는 안된다..

> 계속 헤마다가 알게 되었다, yml 작성을 제대로 해야 한다는 것을..들여쓰기를 일단 잘 해야 한다. 이제 S3에 업로드도 잘 되고, codedeploy 도 동작한다. 

 - provider 가 2개 이상일 때는 `-`를 붙이고, 하위로 동일한 들여쓰기로 작성해나간다. 

```yml
deploy:
  - provider: s3
    access_key_id: $AWS_ACCESS_KEY_ID # Travis repo settings 에 설정된 값
    secret_access_key: $AWS_SECRET_ACCESS_KEY # Travis repo settings 에 설정된 값
    bucket: sixstar6-aws-build # s3 버킷
    region: ap-northeast-2
    skip_cleanup: true
    acl: private # zip 파일 접근을 private 으로
    local_dir: deploy # before_deploy 에서 생성한 디렉토리
    wait-until-deployed: true
  - provider: codedeploy
    access_key_id: $AWS_ACCESS_KEY_ID
    secret_access_key: $AWS_SECRET_ACCESS_KEY
    bucket: sixstar6-aws-build
    key: springboot-ws-started.zip
    bundle_type: zip
    application: springboot-ws-started
    deployment_group: springboot-ws-started-group
    region: ap-northeast-2
    wait-until-deployed: true
```

 - appspec.yml 수정

```yml
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user/app/step2/zip
    overwrite: yes

permissions:
  - object: /
    pattern: "*"
    owner: ec2-user
    group: ec2-user

hooks:
  ApplicationStart:
    - location: deploy.sh
      timeout: 60
      runas: ec2-user
``` 

 - permissions : CodeDeploy에서 EC2 서버로 넘겨준 파일을 모두 ec2-user 권한을 갖도록 한다 
 - hooks : deploy.sh 를 ec2-user 권한으로 실행하게 한다. 타임아웃 60초내에 수행되지 못하면 실패한다. 

## 무중단 배포

 - 무중단 배포 방식
   1. AWS에서 블루그린 무중단 배포
   2. 도커를 이용한 웹서비스 무중단 배포

 - 책에서는 엔진엑스를 사용한다. 엔진엑스는 웹서버, 리버스 프록시, 캐싱, 로드 밸런싱, 미디어 스트리밍등을 위한 오픈소스 소프트웨어이다. 
 - 아파치가 대세였던 자리를 완전히 뺏은 유명한 웹서버이다. 고성능 웹서버이기 때문에 대부분 서비스들이 엔진엑스를 사용한다. 

 - AWS EC2 인스턴스가 하나 더 필요하지 않다. AWS와 같은 클라우드 인프라가 구축되어 있지 않아도 사용할 수 있는 범용적인 방법이다. 
 - EC2 혹은 리눅스 서버에 엔진엑스 1대와 스프링 부트 Jar를 2대 사용하는 것이다. 
 - 엔진엑스는 80, 443 포트를 할당, 스프링부트1은 8081 , 스프링부트2 는 8082를 사용한다. 

1. 엔진엑스는 요청을 스프링부트 1에게 전달한다. 
2. 배포시 스프링부트2로 배포한다. 
3. 배포가 끝나고, 스프링부트2가 정상구동중이면 nginx reload 명령어를 통해 8081대신 8082를 바라보도록 한다. 
4. 이후 배포를 진행할때는 스프링부트 1로 배포한다. 

> 엔진엑스 설치와 스프링부트 연동

`sudo yum install nginx`

- 앗,, 설치가 되지 않는다. 

```xml
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
No package nginx available.
Error: Nothing to do


nginx is available in Amazon Linux Extra topic "nginx1"

To use, run
# sudo amazon-linux-extras install nginx1

Learn more at
https://aws.amazon.com/amazon-linux-2/faqs/#Amazon_Linux_Extras
```

- aws 에서 제공해주는 걸 사용해야 한다.  `sudo amazon-linux-extras install nginx1`

`sudo service nginx start`

 => Redirecting to /bin/systemctl start nginx.service

 > 보안그룹 추가

  - 엔진엑스의 포트번호를 보안그룹에 추가한다. 
    : EC2 -> 보안그룹 -> EC2 보안그룹선택 -> 인바운드 편집

![image1](/assets/images/page11/img65.png)

 - 브라우저에서 8080을 빼고 접속하면 아래와 같이 엔진엑스 웰컴화면이 나온다. 

![image1](/assets/images/page11/img66.png)

> 엔진엑스와 스프링부트 연동

 - sudo vim /etc/nginx/nginx.conf
 - 빨간 박스 안에 내용을 추가한다. 

 ![image1](/assets/images/page11/img67.png)

  - proxy_pass http://localhost:8080 => 엔진엑스로 요청이 오면, localhost:8080으로 전달한다. 
  - proxy_set_header 는 실제 요청데이터를 header 의 각 항목에 할당한다. 

 - 수정이 완료 되었으면, sudo service nginx restart 로 엔진엑스를 재시작한다. 

## 무중단 배포 스크립트 만들기 

 - ProfileController 를 만든다. 배포시 8081을 쓸지 8082를 쓸지 판단하는 기준이 된다. 

```java
@RequiredArgsConstructor
@RestController
public class ProfileController {
    private final Environment env;

    @GetMapping("/profile")
    public String profile(){
        List<String> profiles = Arrays.asList(env.getActiveProfiles());// 현재 실행중인 ActiveProfile 을 모두 가져옴.
        List<String> realProfiles = Arrays.asList("real", "real1", "real2");

        String defaultProfile = profiles.isEmpty() ? "default" : profiles.get(0);

        return profiles.stream()
                .filter(realProfiles::contains)
                .findAny()
                .orElse(defaultProfile);

    }
}
```

- Test Code
- Test에는 스프링 환경이 필요하지 않다. 

```java
@Test
    public void real_profile이_조회(){
        //given
        String expectedProfile = "real";
        MockEnvironment env = new MockEnvironment();
        env.addActiveProfile(expectedProfile);

        ProfileController controller = new ProfileController(env);

        //when
        String profile = controller.profile();

        //then
        assertThat(profile).isEqualTo(expectedProfile);

    }

    @Test
    public void active_profile이없으면_default가조회(){
        //given
        String expectedProfile = "default";
        MockEnvironment env = new MockEnvironment();

        ProfileController controller = new ProfileController(env);

        //when
        String profile = controller.profile();

        //then
        assertThat(profile).isEqualTo(expectedProfile);

    }
```

 - real1, real2 profile 생성

 - application-real1.properties

```java
server.port=8081
spring.profiles.include=real-db
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
```

 - application-real2.properties

```java
`server.port=8082
spring.profiles.include=real-db
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
```

> 엔진엑스 설정 수정

- 무중단 배포의 핵심은 엔진엑스 설정이다. 배포때마다 엔진엑스의 프록시 설정이 순식간에 교체된다. 여기서 프록시 설정이 교체될 수 있도록 설정을 추가한다. 
엔진엑스 설정이 모여있는 /etc/nginx/conf.d/ 에 service-url.inc 라는 파일을 생성한다. 
- conf.d 에서 .d는 directory의 d이다. 기존 단일 설정파일과의 이름충돌을 위해, 여러 설정 파일을 디렉터리로서 묶는다는 의미로 .d를 붙였다.

`sudo vim /etc/nginx/conf.d/service-url.inc`

- 그리고 다음 코드를 입력한다. 

`set $service_url http://127.0.0.1:8080;`

- 저장하고 종료한뒤 nginx.conf 파일을 수정한다. 

`sudo vim /etc/nginx/nginx.conf`

- location / 부분을 찾아 다음과 같이 변경한다. 

```java
include /etc/nginx/conf.d/service-url.inc;

location / {
  proxy_pass $service_url;
}
```

- 저장하고 종료한뒤 재시작한다. 

`sudo service nginx restart`

## 배포 스크립트들 작성

> step3 디렉토리 생성

- mkdir ~/app/step3 && mkdir ~/app/step3/zip
- 무중단 배포는 앞으로 step3를 사용한다. appspec.yml을 step3로 배포되도록 수정한다. 

```yml
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user/app/step3/zip
    overwrite: yes
```

 - 무중단 배포에 총 5개의 스크립트 필요
 1. stop.sh : 기존에 연결되어 있지 않지만, 실행중이던 스프링부트 종료
 2. start.sh : 배포할 신규버전 스프링 부트 프로젝트를 stop.sh로 종료한 'profile'로 실행
 3. health.sh : 'start.sh'로 실행시킨 프로젝트가 정상적으로 실행됐는지 체크
 4. switch.sh: 엔진엑스가 바라보는 스프링부트를 최신버전으로 변경
 5. profile.sh: 앞선 4개 스크립트 파일에서 공용으로 사용할 'profile'과 포트 체크 로직

> 배포 후 로그확인 

`tail -f /opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log`


> 아래 절차대로 수행된다.  

1. 현재 real2 가 떠있음, 엔진엑스는 8082와 연결되어 있다.

2. IDEL_PORT 는 8081 이 되므로, 8081포트로 떠 있는 서비스를 kill 한다. 

3. IDEL_PROFILE 은 real1 이므로 Dspring.profiles.active=real1 으로 스프링부트를 실행한다. 

4. 스프링 부트가 정상적으로 구동하였는지 체크한다. 

5. 정상적으로 구동되었으면, 엔진엑스를 8081과 연결한다. 

> profile.sh

- IDEL 한 port 서비스를 찾는다.  

```sh
#!/usr/bin/env bash

# bash 는 return value 가 안되나 *제일 마지막줄에 echo로 해서 결과 출력*후 그 값을 ($(find_idle_profile))사용한다. 중간에 echo 를 사용하면 안된다.
# 쉬고 있는 profile 찾기: real1이 사용중이면 real2가 쉬고 있꼬, 반대면 real1이 쉬고 있음
function find_idle_profile()
{
    RESPONSE_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost/profile)

    if [ ${RESPONSE_CODE} -ge 400 ] # 400 보다 크면 (즉, 40x/50x 에러 모두 포함)
    then
        CURRENT_PROFILE=real2
    else
        CURRENT_PROFILE=$(curl -s http://localhost/profile) # 현재 엔진엑스와 연결되어 있는 profile 체크, real1 또는 real2
    fi

# 현재 profile 이 real1 이면, IDEL_PROFILE은 real2 가 된다. 
    if [ ${CURRENT_PROFILE} == real1 ]
    then
      IDLE_PROFILE=real2
    else
      IDLE_PROFILE=real1
    fi

    echo "${IDLE_PROFILE}"
}

# 쉬고 있는 profile의 port 찾기
function find_idle_port()
{
    IDLE_PROFILE=$(find_idle_profile)

    if [ ${IDLE_PROFILE} == real1 ]
    then
      echo "8081"
    else
      echo "8082"
    fi
}

```

> stop.sh

- 스크립트 실행할 때 명령어를 인자로 받는데, 그 인자의 경로를 해석해서 파일의 위치를 구할 수 있다. 
- ABSPATH=$(readlink -f $0), 에서 $0 은 명령어의 첫번째인자를 말한다. bash ./stop.sh 에서 $0은 ./stop.sh 을 말한다. 

```sh
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH) # 현재 stop.sh 가 속한 경로를 찾는다.
source ${ABSDIR}/profile.sh # profile.sh 를 import 한다.

IDLE_PORT=$(find_idle_port)

echo "> $IDLE_PORT 에서 구동중인 애플리케이션 pid 확인"
IDLE_PID=$(lsof -ti tcp:${IDLE_PORT}) # port 로 pid 를 구한다. 

if [ -z ${IDLE_PID} ]
then
  echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
else
  echo "> kill -15 $IDLE_PID"
  kill -15 ${IDLE_PID}
  sleep 5
fi
```

> start.sh

- deploy.sh 과 비슷하다. 다만 active 값을 구분하여, 스프링부트를 구동한다. 

```sh
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

REPOSITORY=/home/ec2-user/app/step3
PROJECT_NAME=freelec-springboot2-webservice

echo "> Build 파일 복사"
echo "> cp $REPOSITORY/zip/*.jar $REPOSITORY/"

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 새 어플리케이션 배포"
JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME

echo "> $JAR_NAME 실행"

IDLE_PROFILE=$(find_idle_profile)

echo "> $JAR_NAME 를 profile=$IDLE_PROFILE 로 실행합니다."
nohup java -jar \
    -Dspring.config.location=classpath:/application.properties,classpath:/application-$IDLE_PROFILE.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
    -Dspring.profiles.active=$IDLE_PROFILE \
    $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```

> health.sh

- 스프링부트가 정상적으로 구동하였는지 체크하여, 정상일 경우, 엔진엑스와 연결되는 포트를 변경한다. 

```sh
#!/usr/bin/env bash

# 엔진엑스와 연결되지 않은 포트로 스프링 부트가 잘 수행되었는지 체크

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh
source ${ABSDIR}/switch.sh

IDLE_PORT=$(find_idle_port)

echo "> Health Check Start!"
echo "> IDLE_PORT: $IDLE_PORT"
echo "> curl -s http://localhost:$IDLE_PORT/profile "
sleep 10

for RETRY_COUNT in {1..10}
do
  RESPONSE=$(curl -s http://localhost:${IDLE_PORT}/profile)
  UP_COUNT=$(echo ${RESPONSE} | grep 'real' | wc -l)

  if [ ${UP_COUNT} -ge 1 ]
  then # $up_count >= 1 ("real" 문자열이 있는지 검증)
      echo "> Health check 성공"
      switch_proxy
      break
  else
      echo "> Health check의 응답을 알 수 없거나 혹은 실행 상태가 아닙니다."
      echo "> Health check: ${RESPONSE}"
  fi

  if [ ${RETRY_COUNT} -eq 10 ]
  then
    echo "> Health check 실패. "
    echo "> 엔진엑스에 연결하지 않고 배포를 종료합니다."
    exit 1
  fi

  echo "> Health check 연결 실패. 재시도..."
  sleep 10
done
```

> switch.sh

```sh
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

function switch_proxy() {
    IDLE_PORT=$(find_idle_port)

    echo "> 전환할 Port: $IDLE_PORT"
    echo "> Port 전환"
    echo "set \$service_url http://127.0.0.1:${IDLE_PORT};" | sudo tee /etc/nginx/conf.d/service-url.inc

    echo "> 엔진엑스 Reload"
    sudo service nginx reload
}
```

> 빌드 버전 자동 갱신

- build.gradle 에 version 을 Groovy 언어를 이용하여 시간에 따라 변경되도록 한다. 

`version = '1.0.1-SNAPSHOT-'+new Date().format("yyyyMMddHHmmss")`

# Reference
> - 스프링부트와 AWS 혼자 구현하는 웹서비스 ( by 이동욱)

