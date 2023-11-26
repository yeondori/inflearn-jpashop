# 섹션 6. 주문 도메인 개발

비즈니스 로직이 얽혀 있어 매우 중요한 파트이다!
 
구현 기능
- 상품 주문
- 주문 내역 조회
- 주문 취소


## 주문, 주문상품 엔티티 개발

```java
//==생성 메서드==//
    public static Order createOrder(Member member, Delivery delivery, OrderItem... orderItems) {
        Order order = new Order();
        order.setMember(member);
        order.setDelivery(delivery);
        for (OrderItem orderItem : orderItems) {
            order.addOrderItem(orderItem);
        }
        order.setStatus(OrderStatus.ORDER);
        order.setOrderDate(LocalDateTime.now());
        return order;
    }
```

주문, 생성에 관한 비즈니스 로직을 생성 메서드에 응집해둔다!
마찬가지로 OrderItem에도 생성 메서드를 추가해준다.

## 주문 리포지토리 개발
간단히 save와 findOne 로직을 추가한다. 검색과 관련해서는 추후에 다룬다.

## 주문 서비스 개발

```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;
    private final MemberRepository memberRepository;
    private final ItemRepository itemRepository;

    //주문
    @Transactional
    public Long order(Long memberId, Long itemId, int count) {

        //엔티티 조회
        Member member = memberRepository.findOne(memberId);
        Item item = itemRepository.findOne(itemId);

        //배송 정보 생성
        Delivery delivery = new Delivery();
        delivery.setAddress(member.getAddress());

        //주문상품 생성
        OrderItem orderItem = OrderItem.createOrderItem(item, item.getPrice(), count);

        //주문 생성
        Order order = Order.createOrder(member, delivery, orderItem);
        
        //주문 저장
        orderRepository.save(order);
        
        return order.getId();
    }
}

```

주문 메서드를 살펴보면 orderRepository에서만 save를 하고 있다. 이는 연관관계 매핑에서 cascade를 CascadeType.ALL로 설정했기 때문이다.
order를 persist할 때, orderItem과 delivery가 함께 persist 되는 것이다!

cascade 범위는 참조하는 주인이 private 오너인 경우만 사용해야 한다. 즉, life cycle이 동일하게 관리되고 다른 것이 참조할 수 없는 private한 경우에 사용한다!
Delivery나 OrderItem을 다른 엔티티에서도 참조하고 있다면 cascade를 사용하면 안되고 별도의 Repository를 생성해주어야 한다. 

참고로 생성 방식을 통일하고 제어하기 위해 생성자를 protected로 제한해주는 것이 좋다.
@NoArgsConstructor(access = AccessLevel.PROTECTED) 를 사용하면 생성 메서드를 이용해 생성하도록 간단히 제어할 수 있다!

예제에서는 단순히 주문 상품을 하나만 넣을 수 있도록 했다.

비즈니스 로직이 대부분 엔티티에 있음을 주목하자! 이런 스타일을 **도메인 모델 패턴** 이라고 한다.      
이와 반대로 비즈니스 로직이 대부분 서비스 계층에서 처리될 때는 **트랜젝션 스크립트 패턴** 이라고 한다.   
문맥에 따라 trade-off가 있으니 잘 생각해보자. 그리고 프로젝트 안에서도 두 패턴이 양립할 수 있다. 

## 주문 기능 테스트

참고로 사실 현재의 예제는 jpa가 동작하는 것을 보여주기 위한 목적의 테스트이며 좋은 테스트라고 볼 수는 없다.
좋은 테스트는 DB나 Spring,  dependency 없이 순수히 메서드에 대해 단위 테스트를 하는 것이다. 

## 주문 검색 기능 개발

```java
public List<Order> findAll(OrderSearch orderSearch) {
        return em.createQuery("select o from Order o join o.member m" +
                        "where o.status = :status" +
                        "and m.name like :name", Order.class)
                .setParameter("status", orderSearch.getOrderStatus())
                .setParameter("name", orderSearch.getMemberName())
                .setMaxResults(1000)
                .getResultList();
    }
```

위의 코드는 status와 member가 모두 존재해야 가능하며 jqpl을 동적으로 생성해야 하는 경우 다음과 같이 작성해야 한다. 

```java
public List<Order> findAllByString(OrderSearch orderSearch) {

        String jpql = "select o from Order o join o.member m";
        boolean isFirstCondition = true;

        //주문 상태 검색
        if (orderSearch.getOrderStatus() != null) {
        jpql += " where";
        isFirstCondition = false;
        } else {
        jpql += "and";
        }
        jpql += " o.status = :status";

        //회원 이름 검색
        if (StringUtils.hasText(orderSearch.getMemberName())) {
        jpql += " where";
        isFirstCondition = false;
        } else {
        jpql += "and";
        }
        jpql += " m.name like :name";

        TypedQuery<Order> query = em.createQuery(jpql, Order.class)
        .setMaxResults(1000);

        if (orderSearch.getOrderStatus() != null) {
        query = query.setParameter("status", orderSearch.getOrderStatus());
        }
        if (StringUtils.hasText(orderSearch.getMemberName())) {
        query = query.setParameter("name", orderSearch.getMemberName());
        }

        return query.getResultList();
        }
```

그러나 보다 싶이 매우 복잡하고 번거로우며 오타로 인한 버그를 찾기도 어렵다.

두번째 방법으로는 jpa가 제공하는 표준 동적 쿼리를 빌드해주는 것을 사용하는 방식이다.

```java
public List<Order> findAllByCriteria(OrderSearch orderSearch) {
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<Order> cq = cb.createQuery(Order.class);
        Root<Order> o = cq.from(Order.class);
        o.join("member", JoinType.INNER);

        List<Predicate> criteria = new ArrayList<>();

        //주문 상태 검색
        if (orderSearch.getOrderStatus() != null) {
            Predicate status = cb.equal(o.get("status"), orderSearch.getOrderStatus());
            criteria.add(status);
        }
        //회원 이름 검색
        if (StringUtils.hasText(orderSearch.getMemberName())) {
            Predicate name = 
                    cb.like(o.get("name"), "%" + orderSearch.getMemberName() + "%");
            criteria.add(name);
        }

        cq.where(cb.and(criteria.toArray(new Predicate[criteria.size()])));
        TypedQuery<Order> query = em.createQuery(cq).setMaxResults(1000);

        return query.getResultList();
}
```

jpql을 java 코드로 작성해주는 장점이 있으나 쿼리가 명시적이지 않고 유지보수가 매우 어렵다는 치명적인 단점이 있다.   
따라서 jpa 표준 스택에 있으나 운영에서 사용하기엔 무리가 있다.

결론은 **query DSL** 을 사용하자!