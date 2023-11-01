# inflearn-jpashop
김영한님의 인프런 강의 [[실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발 강의](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1/dashboard) 

# 섹션 1. 프로젝트 환경설정

## 프로젝트 환경
![img](https://img.shields.io/badge/gradle-02303A?style=for-the-badge&logo=gradle&logoColor=white) ![img](https://img.shields.io/badge/java-007396?style=for-the-badge&logo=java&logoColor=white) ![Spring Boot](https://img.shields.io/badge/springboot-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
- **Project**: Gradle
- **Language**: Java
- **Spring Boot**: 3.1.5
- **Java**: 17
- **Dependencies**: Lombok, Spring Web, Thymeleaf, Spring Data JPA, H2 Database

++ lombok 플러그인 설치 후 Settings에서 annotation processors에서 enable annotation processing 체크하기

## 동적 페이지와 정적 페이지

정적 페이지인 index.html과 동적 페이지인 hello.html을 생성해본다. 

참고로 `implementation 'org.springframework.boot:spring-boot-devtools'` 를 라이브러리에 추가해주면 html 파일을 Recompile하는 것만으로도 변동사항 확인이 가능하다!

## H2 DB 연동

H2를 설치한 뒤 DB를 생성하고, 이를 프로젝트와 연동하기 위해 다음의 작업을 수행한다.

우선 편리성을 위해 application.properties 파일을 삭제하고 application.yml 파일을 생성해주었다.

H2 DB를 연동하기 위해서는 application.yml에 다음과 같이 datasource 설정을 해주면 된다.

```yml
spring:
    datasource:
        url: jdbc:h2:tcp://localhost/~/inflearn-jpashop;MVCC=TRUE
        username: sa
        password:
        driver-class-name: org.h2.Driver

    jpa:
        hibernate:
          ddl-auto: create #실행 시점에 정보를 모두 지우고 다시 생성
        properties:
          hibernate:
            show_sql: true #SystemOut에 출력된다. 운영 환경에서는 사용하면 안됨!
            format_sql: true

logging:
  level:
    org.hibernate.SQL: debug #hibernate가 생성하는 SQL이 보이도록 하는 옵션! show_sql과는 달리 로거를 통해 찍힌다.
```
참고로 이런 설정들은 [Spring Boot 매뉴얼](https://spring.io/projects/spring-boot#learn)에서 직접 배워야 한다 ^_^

