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

## Test

데이터 베이스 연동을 확인하기 위해 Member Class와 MemberRepository를 만든 뒤 MemberRepositoryTest를 통해 검증한다.
여기에서 발생하는 에러는 [github 블로그]()에서 다루고 있다. 

**Member**
```java
package jpabook.jpashop;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import lombok.Getter;
import lombok.Setter;

@Entity
@Getter @Setter
public class Member {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;

}
```

**MemberRepository**
```java
package jpabook.jpashop;

import jakarta.persistence.EntityManager;
import jakarta.persistence.PersistenceContext;
import org.springframework.stereotype.Repository;

@Repository
public class MemberRepository {

    @PersistenceContext
    private EntityManager em;

    public Long save(Member member) {
        em.persist(member);
        return member.getId();
    }

    public Member find(Long id) {
        return em.find(Member.class, id);
    }
}
```

**MemberRepositoryTest**
```java
package jpabook.jpashop;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Transactional;
import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest
public class MemberRepositoryTest {

    @Autowired MemberRepository memberRepository;

    @Test
    @Transactional // test에 있는 경우 test 이후 rollback된다.
    public void testMember() throws Exception {
        //given
        Member member = new Member();
        member.setUsername("memberA");

        //when
        Long saveId = memberRepository.save(member);
        Member findMember = memberRepository.find(saveId);

        //then
        assertThat(findMember.getId()).isEqualTo(member.getId());
        assertThat(findMember.getUsername()).isEqualTo(member.getUsername());
        assertThat(findMember).isEqualTo(member);
    }
}
```

참고로 equals HashCode 등이 구현되지 않은 상태에서 `assertThat(findMember).isEqualTo(member);` 를 수행한다면 정상적으로 작동한다.
같은 transaction 안에서 저장, 조회가 되므로 같은 영속성 컨텍스트에 있고 식별자가 같으므로 같은 것으로 인식되기 때문이다. 

참고로 터미널에서 `./gradlew clean build` 로 깔끔히 지우고 다시 빌드한 뒤, 라이브러리에 들어가 `java -jar `로 jar 파일을 실행시켜주어도 콘솔에서 실행이 가능하다!

또한, 쿼리 파라미터를 남기고 싶은 경우에 application.yml에서 logging level에 `org.hibernate.type: trace`를 추가하면 trace 로그에서 파라미터 확인이 가능하다.
조금 더 편리하게 보고 싶다면 build.gradle에 `implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.6'`를 추가해 외부 라이브러리를 사용하는 방법도 있다. 
그러나 운영 배포 시에는 성능 저하의 원인이 될 수 있으므로 성능 테스트를 해봐야 한다!