# 섹션 4. 회원 도메인 개발

## MemberRepository

- @Repository 
  - 스프링 빈으로 등록되어 Component Scan의 대상이 된다


- @PersistenceContext
  - 스프링이 JPA의 EntityManager를 주입해준다. 
  - 만약 EntityManagerFactory를 직접 주입받고 싶다면 @PersistenceUnit 을 추가하면 된다.

## MemberService

- @Service
  - 스프링 빈으로 등록되어 Component Scan의 대상이 된다


- @Transactional
  - 데이터 변경은 트랜잭션 안에서 이뤄져야 한다.
  - javax와 spring 중 spring을 사용하는 걸 권장한다. 사용할 수 있는 옵션이 더 많다.
  - 메서드에서 @Transactional(readOnly = true) 옵션을 사용하면 조회시 성능을 최적화한다. 
  - 조회가 많다면 Service에 readOnly = true 옵션을 달고, 변경이 있는 부분에 Transactional 어노테이션을 붙이는 방식을 권장한다.


- 의존성 주입
  - 현재는 Field 인젝션으로 의존성을 주입해주었다. 이는 변경에 불리하다는 단점이 있다.
  - Setter 인젝션은 메서드를 통해 Mock등을 주입해 줄 수 있어 편리하나 실제 애플리케이션 동작 시에 변경할 일은 거의 없다.
  - 가장 추천하는 방식은 **생성자 인젝션** 이다. 생성 시점에 명확히 알 수 있다.   
    스프링은 생성자가 하나만 있을 때 자동으로 Autowired를 해준다. 필드를 final로 하는 것을 추천한다.  
    이때 @AllArgsConstructor 는 필드들의 생성자를 만들어준다. **@RequiredArgsConstructor** 는 final 필드만 가지고 생성자를 만들어 주어 더 좋다!
   


- validateDuplicateMember
  - 동시에 DB에 등록한다면 이 로직을 같이 통과할 수 있다.
  - 멀티 스레드 상황을 고려해 최후의 방어를 해주는 게 좋다
  - DB에 Member name을 unique 제약조건을 주는 것이 더욱 안전하다.

## 회원 기능 테스트
- @RunWith(SpringRunner.class), @SpringBootTest
  - JUnit을 실행할 때 스프링과 엮어서 실행하고 싶은 경우 (JUnit4)


- @Transactional
  - 테스트에 이 어노테이션이 있는 경우 기본적으로 Rollback을 수행한다.


- @Test(expected = IllegalStateException.class) 
  - try-catch 없이 보다 더 간단히 예외를 잡을 수 있다.


- Test DB
  - 기본적으로 Test를 따로 격리해주는 것이 좋다.
  - test 폴더에 resources/application.yml을 생성해주면 이 application.yml만을 참고한다.
  - 여기서 datasource의 url을 `jdbc:h2:tcp://localhost/~/inflearn-jpashop` -> `jdbc:h2:mem:test` 로 변경하면 메모리 모드로 동작한다!
  - 그러나 사실 스프링부트는 별도의 설정이 없다면 메모리모드로 동작하므로 생략해주어도 된다.
  - 그렇지만 테스트는 메인과 별도로 applicatino.yml을 분리해주는 것이 좋다.