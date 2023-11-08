# 섹션 2. 도메인 분석 설계

## 요구사항 분석

- 회원 기능: 회원 등록/조회  


- 상품 기능: 상품 등록/수정/조회  


- 주문 기능: 상품 등록/수정/조회


- 기타 요구사항 
  - 상품은 재고 관리가 필요
  - 상품 종류: 도서, 음반, 영화
  - 상품을 카테고리로 구분 가능
  - 상품 주문 시 배송 정보 입력

## 도메인 모델과 테이블 설계 
[실전! 스프링 부트와 JPA 활용1 강의 자료]
![image](https://github.com/yeondori/inflearn-jpashop/assets/93027942/35ea49c1-fc6f-4760-8525-38d036973075)
![image](https://github.com/yeondori/inflearn-jpashop/assets/93027942/09c52b56-5b85-4557-8150-1849d02d81d4)

학습 목적으로 JPA에서 표현할 수 있는 모든 관계를 활용하는 방식의 설계이다.
그러나 실무에서 다대다는 [JPA 기본편](https://yeondori.github.io/posts/jpa-basic-06/#%EB%8B%A4%EB%8C%80%EB%8B%A4-nn)에서 언급한 것처럼 사용하지 않는 것이 좋다.
양방향 연관관계를 사용하는 것도 지양하는 것이 좋으나 학습을 위해 추가되어 있다.  

> 회원과 주문에서, 회원이 주문을 하므로 ordersList를 생각할 수 있으나 시스템 자체에서는 회원과 주문을 동일선상에 두어야 한다.
 회원을 통해 주문을 하는 것이 아니고 주문을 생성할 때 회원이 필요하다고 생각해야 한다.  
 쿼리 시에도 주문 내역이 필요한 경우 멤버의 orderList를 살펴보는 게 아니라 주문에서 필터링 조건으로 멤버를 찾는 것이 바람직한 것 처럼 말이다. 

객체에서는 양쪽 다 컬렉션을 가져 다대다를 만들 수 있으나, 관계형 데이터베이스에서는 매핑테이플로 다대다 관계를 일대다, 다대일 관계로 풀어야 한다.

## 엔티티 클래스 개발

주어진 설계대로 엔티티를 생성하고 연관관계를 매핑한다.  

- 강의에서는 Getter와 Setter를 모두 사용하나 이론적으로는 둘다 제공하지 않고 필요한 메서드만 제공하는 것을 지향한다. 그러나 실무에서는 편의상 Getter는 열어두고 Setter는 닫아둔다.   

  >  Setter를 막 열어두면 엔티티 변경 추적이 어려워질 수 있으므로 이 대신에 변경 지점이 명확하도록 변경을 위한 비즈니스 메서드를 별도로 제공하는 것이 좋다.

- 연관관계 설정, @Enumerated(EnumType.STRING) 어노테이션 설정은 특히나 잊지 않도록 주의하자!  


- 다대다의 경우 @JoinColumn이 아닌 @JoinTable로 테이블을 매핑해야 한다. 앞서 말했듯 다대다는 실무에서 사용하는 걸 지양하자.


- id의 컬럼명을 `table명_id`로 설정한 이유는 어디 소속인지 명확히 밝혀주기 위해서이다. 객체는 명확히 타입이 있으나 테이블은 타입이 없으므로 구분을 위해 명시해주는 것이 좋다.


- 값 타입은 기본적으로 Immutable해야 한다. 생성 시점에 값을 세팅하고 변경할 수 없도록 하는 것이 좋다. 
    ```java
    package jpabook.jpashop.domain;
    
    import jakarta.persistence.Embeddable;
    import lombok.Getter;
    
    @Embeddable
    @Getter
    public class Address {
    private String city;
    private String street;
    private String zipcode;
    
        protected Address() {
        }
    
        public Address(String city, String street, String zipcode) {
            this.city = city;
            this.street = street;
            this.zipcode = zipcode;
        }
    }
    
    ```
  JPA 기본 스펙상 프록시나 리플렉션을 허용하기 위해 임베디드 타입은 기본 생성자를 만들어줘야 한다. public 대신 protected로 접근을 제어해주는 것이 좋다.


- 설계한 대로 생성 되었는 지를 수시로 확인해보자! 

## 엔티티 설계시 주의점 

- 가급적 Setter를 사용하지 않는다.  


- 모든 연관관계는 **지연로딩**으로 설정한다.
  > 최적화는 패치 조인이나 엔티티 그래프 기능을 사용하고, 모든 연관관계는 지연로딩으로 설정해야 한다.  
  > 특히나 @ManyToOne, @OneToOne 은 default값이 EAGER이므로 반드시 설정해줘야 한다.

  
- 컬렉션은 필드에서 초기화하자.
  > 하이버네이트는 엔티티를 영속화할 때 컬렉션을 감싸서 내장 컬렉션으로 변경한다.   
  > 임의의 메서드에서 컬렉션을 잘못 생성하면 하이버네이트 내부 매커니즘에 문제가 발생할 수 있다.   
  > 따라서 컬렉션을 가급적이면 변경하지 않는 것이 가장 안전하다.  
  
- 테이블, 컬럼명 생성 전략
  - 방식을 변경하고 싶다면 `spring.jpa.hibernate.naming.physical-strategy` 로 변경 가능하다
  - 스프링 부트 신규 설정은 다음과 같다.
    - 카멜 케이스 -> 언더스코어(_)
    - 점(.) -> 언더스코어(_)
    - 대문자 -> 소문자
  - 논리적(명시적으로 적혀있지 않은 경우), 물리적(모든 경우) 적용은 강의 자료를 참고하자.

- cascade 
  ```java
  @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
  private List<OrderItem> orderItems = new ArrayList<>();
  ```
  cascade 옵션을 해두면 엔티티별로 persist를 호출할 필요가 없어진다.

- 연관관계 편의 메서드
  양방향의 경우, 다음과 같이 세팅을 원자적으로 한쪽에서 수행할 수 있도록 메서드를 생성할 수 있다.   
  위치는 핵심적으로 컨트롤하는 쪽에 두는 것이 좋다. 

    ```java
    public void setMember(Member member) {
            this.member = member;
            member.getOrders().add(this);
        }
  
        public void addOrderItem(OrderItem orderItem) {
            orderItems.add(orderItem);
            orderItem.setOrder(this);
        }
      
        public void setDelivery(Delivery delivery) {
            this.delivery = delivery;
            delivery.setOrder(this);
        }
    ```