# 포토그램 - 인스타그램 클론 코딩

### 배포 프로세스
- AWS 회원가입
- github 소스코드 다운받기 (war 모드로 변경함)
- profile 세팅
- RDS 만들기
- 테이블 생성하기
- Elastic Beanstalk 만들기
- RDS 포트 3306 개방하기 (내IP와 Elastic Beanstalk 보안그룹)
- Elastic Beanstalk war 배포하기

### 빌드하기
mvn clean package

### 테이블 생성
```sql
create database photogram;

create table Comment (
  id integer not null auto_increment,
  content varchar(100) not null,
  createDate datetime(6),
  imageId integer,
  userId integer,
  primary key (id)
) ENGINE=InnoDB DEFAULT CHARSET=UTF8MB4;


create table Image (
  id integer not null auto_increment,
  caption varchar(255),
  createDate datetime(6),
  postImageUrl varchar(255),
  userId integer,
  primary key (id)
) ENGINE=InnoDB DEFAULT CHARSET=UTF8MB4;


create table Likes (
  id integer not null auto_increment,
  createDate datetime(6),
  imageId integer,
  userId integer,
  primary key (id)
) ENGINE=InnoDB DEFAULT CHARSET=UTF8MB4;


create table Subscribe (
  id integer not null auto_increment,
  createDate datetime(6),
  fromUserId integer,
  toUserId integer,
  primary key (id)
) ENGINE=InnoDB DEFAULT CHARSET=UTF8MB4;


create table User (
  id integer not null auto_increment,
  bio varchar(255),
  createDate datetime(6),
  email varchar(255) not null,
  gender varchar(255),
  name varchar(255) not null,
  password varchar(255) not null,
  phone varchar(255),
  profileImageUrl varchar(255),
  role varchar(255),
  username varchar(100),
  website varchar(255),
  primary key (id)
) ENGINE=InnoDB DEFAULT CHARSET=UTF8MB4;


alter table Likes 
 add constraint likes_uk unique (imageId, userId);

alter table Subscribe 
 add constraint subscribe_uk unique (fromUserId, toUserId);

alter table User 
 add constraint UK_jreodf78a7pl5qidfh43axdfb unique (username);

alter table Comment 
 add constraint FKm1kgtoxiwl6jebkoqxesmh20k 
 foreign key (imageId) 
 references Image (id);

alter table Comment 
 add constraint FKr4r2wh1b3rucuaxui9lkjwjlu 
 foreign key (userId) 
 references User (id);

alter table Image 
 add constraint FKgrmt25snbia9s3sxls7gn3tvl 
 foreign key (userId) 
 references User (id);

alter table Likes 
 add constraint FKdrmcrl980hncyhnurju8nm5dy 
 foreign key (imageId) 
 references Image (id);

alter table Likes 
 add constraint FK1f0ppyupbg6s2v5i9h74rglto 
 foreign key (userId) 
 references User (id);

alter table Subscribe 
 add constraint FK9dl9afu79ab4cxwbbgif6t6ie 
 foreign key (fromUserId) 
 references User (id);

alter table Subscribe 
 add constraint FKl3r4mww8oeu08s3mqyq138qp3 
 foreign key (toUserId) 
 references User (id);
```

### 인메모리 DB로 테스트 하는 yml, maven

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

```yml
server:
  port: 8080
  servlet:
    context-path: /
    encoding:
      charset: utf-8
      enabled: true
    
spring:
  mvc:
    view:
      prefix: /WEB-INF/views/
      suffix: .jsp
      
  datasource:
    url: jdbc:h2:mem:test;MODE=MySQL
    driver-class-name: org.h2.Driver
    username: sa
    password: 
  h2:
    console:
      enabled: true
    
  jpa:
    open-in-view: true
    hibernate:
      ddl-auto: create # update <- 서버 데이터 유지 / create <- 사라짐 / none <- 아무것도 변경 못하게... / create-drop도 있는데 몰라도 된다고...
      naming:
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
    show-sql: true
      
  servlet:
    multipart: # multipart 스타일로 사진을 받겠다.
      enabled: true # true <- 사진을 받겠다.
      max-file-size: 2MB # 사진 최대 용량은 2MB가 넘지 않도록 제한

  security:
    user:
      name: test
      password: 1234   

file: # 내가 만든 키값
  path: C:/workspace/upload/ # 업로드된 사진 저장할 공간(폴더)
```

### STS 툴에 세팅하기 - 플러그인 설정
- https://blog.naver.com/getinthere/222322821611

### 의존성

- Sring Boot DevTools
- Lombok
- Spring Data JPA
- MariaDB Driver
- Spring Security
- Spring Web
- oauth2-client

```xml
<!-- 시큐리티 태그 라이브러리 -->
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-taglibs</artifactId>
</dependency>

<!-- JSP 템플릿 엔진 -->
<dependency>
	<groupId>org.apache.tomcat</groupId>
	<artifactId>tomcat-jasper</artifactId>
	<version>9.0.43</version>
</dependency>

<!-- JSTL -->
<dependency>
	<groupId>javax.servlet</groupId>
	<artifactId>jstl</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### 데이터베이스

```sql
create user 'cos'@'%' identified by 'cos1234';
GRANT ALL PRIVILEGES ON *.* TO 'cos'@'%';
create database photogram;
```

### yml 설정

```yml
server:
  port: 8080
  servlet:
    context-path: /
    encoding:
      charset: utf-8
      enabled: true
    
spring:
  mvc:
    view:
      prefix: /WEB-INF/views/
      suffix: .jsp
      
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    url: jdbc:mariadb://localhost:3306/costa?serverTimezone=Asia/Seoul
    username: costa
    password: costa1234
    
  jpa:
    open-in-view: true
    hibernate:
      ddl-auto: update
      naming:
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
    show-sql: true
      
  servlet:
    multipart:
      enabled: true
      max-file-size: 2MB

  security:
    user:
      name: test
      password: 1234   

file:
  path: C:/src/springbootwork-sts/upload/
```

### 태그라이브러리

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags"%>
```
