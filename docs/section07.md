# 섹션 7. 웹 계층 개발

## 홈 화면과 레이아웃

@Slf4j 어노테이션을 붙여주면 로그를 확인할 수 있다.

[부트스트랩](https://getbootstrap.com/docs/5.3/getting-started/download/)에서 다운받은 파일을 resources/static 으로 옮겨주면 테마가 적용된다.
참고로 강의에서 사용하는 버전은 [이 곳](https://getbootstrap.com/docs/4.3/getting-started/download/)에서 다운받을 수 있고,
다운없이 header.html을 [다음](https://www.inflearn.com/questions/151764/bootstrap%EC%9D%84-%EC%A7%81%EC%A0%91-%EB%8B%A4%EC%9A%B4%EB%B0%9B%EC%A7%80%EC%95%8A%EA%B3%A0-cdn%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%B4-%EA%B0%84%ED%8E%B8%ED%95%98%EA%B2%8C-%EC%A0%81%EC%9A%A9%ED%95%98%EB%8A%94-%EB%B2%95%EC%9E%85%EB%8B%88%EB%8B%B9) 과 같이 변경해주는 방법도 있다.

## 회원 등록

@NotEmpty 어노테이션을 붙여주면 Spring이 해당 필드를 validate해 비어있다면 오류가 나도록 해준다. validation을 원하는 곳에서 @Valid 어노테이션을 추가한다.

MemberForm의 name 필드에 @NotEmpty 어노테이션을 추가하고 멤버를 create하는 곳에서 다음과 같이 @Valid 어노테이션을 추가해준다. 
참고로 뒤에 Spring의 BindingResult를 추가해주면 오류를 컨트롤러에 넘겨준다.

따라서 다음과 같이 에러 처리를 해줄 수 있다. 

```java
@PostMapping("/members/new")
    public String create(@Valid MemberForm form, BindingResult result) {
        
        if  (result.hasErrors()) {
            return "members/createMemberForm";
        }

        Address address = new Address(form.getCity(), form.getStreet(), form.getZipcode());
        Member member = new Member();
        member.setName(form.getName());
        member.setAddress(address);

        memberService.join(member);
        return "redirect:/";
    }
```

참고로 createMemberForm에서도 에러를 렌더링해주고 있다. 

화면에서 넘어올 때의 validation과 실제 도메인이 원하는 validation이 다를 수 있으므로, MemberForm과 같이 화면에 fit한 form 데이터를 만든 뒤에 
데이터를 받는 것이 좋다. 

## 회원 목록 조회

앞서 말한 대로 요구사항이 정말 간단한 경우에는 폼 없이 엔티티를 그대로 사용해도 된다.  
그러나 실무의 요구사항은 그렇게 단순하지 않다. 엔티티를 폼으로 쓰면 엔티티에 화면에 사용하기 위한 종속적인 기능이 생기고 지저분해질 수 있다.
JPA를 사용할 때는 최대한 dependency없이 핵심 비즈니스 로직만 가질 수 있도록 해야만 애플리케이션이 커져도 유지보수성을 높일 수 있다.

따라서 화면에 맞는 API는 DTO나 Form을 사용하는 것을 권장한다. 엔티티는 최대한 순수하게 유지하자!

현재 회원 목록 기능에서도 members를 그대로 화면에 뿌리고 있으나 실무에서는 DTO나 화면에 맞는 오브젝트로 변환해 반환하는 방식을 권장한다. 

## 상품 등록

예제는 편의상 setter를 사용하고 있지만 실무에서는 setter 대신 static으로 Create할 수 있는 메서드를 클래스 내부에 만들어서 setter없이 설계해주는 것이 바람직하다.

## 상품 목록

회원 목록과 같은 방식으로 진행한다 

## 상품 수정
 
매우 중요한 부분이라 집중해야 한다.  
JPA에서는 **변경 감지를 사용하는 것이 Best Practice** 이다. 

현재는 uri에 id가 들어가 있는데 이는 외부의 수정에 매우 취약하다. 따라서 해당 유저가 맞는지 검증하는 로직을 추가하는 등의 조치를 취하는 것이 적절하다.

## 변경 감지와 병합(merge)

매우매우 중요한 부분이다.

값을 변경하면 JPA가 트랜젝션 커밋 시점에 바뀐 부분을 찾아 update를 날린 뒤 업데이트되는 것이 기본 매커니즘이다. (변경 감지 - dirty checking)
준영속 엔티티란 영속성 컨텍스트가 더는 관리하지 않는 엔티티를 말하는데, 위의 상품수정 예시를 살펴보면 Book의 객체는 새로운 객체이나 이미 id가 세팅되어 있고 DB에 저장되어 있는 상태로 식별자가 정확히 있는 상태에 있다.   
따라서 이를 준영속 엔티티라고 할 수 있는데 문제는 이는 JPA가 관리하지 않는다. 

준영속 엔티티를 수정하는 2가지 방법이 있는데, 다음과 같다.

1. 변경 감지 기능 사용

예를 들어 ItemService에 다음과 같은 메서드를 추가한다. 
    
```java
    @Transactional
    public void updateItem(Long itemId, Book param) {
        Item findItem = itemRepository.findOne(itemId);
        findItem.setPrice(param.getPrice());
        findItem.setName(param.getName());
        ... 
    }
```
트랜젝션 안에서 조회한 findItem은 영속상태에 있으며 Transactional에 의해 트랜젝션되고 flush를 통해 변경값을 update할 수 있게 된다.

2. 병합 사용

병합은 1번을 한줄로 축약한 것이라고 생각하면 간단하다.

merge를 실행하면 파라미터로 넘어온 준영속 엔티티의 식별자 값으로 1차 캐시에서 찾고, 없다면 db에서 조회한다.  
찾은 멤버의 데이터를 파라미터의 데이터로 모두 대치하고, 이를 반환한다. 

파라미터로 넘긴 값은 영속성 컨텍스트에서 변하진 않고, 반환된 값만 영속성 컨텍스트에서 관리된다. 두개는 다르다!

그러나 주의할 점은 병합을 사용하게 되면 모든 값이 변경된다는 것이다. 
merge는 모든 필드를 교체하기 때문에 없는 값은 기존의 값으로 유지되는 것이 아니라 null로 대치된다. 
실무는 복잡하며 merge로 해결하는 것은 매우 위험하고 어려움이 있다. **가급적이면 merge를 사용하지 않는 것이 좋다.**

```java
public void save(Item item) {
    if (item.getId() == null) {
        em.persist(item);
    } else {
        em.merge(item);  //업데이트와 비슷
    }
}
```

또한, setter보다는 값을 변경하는 의미있는 메서드를 만들고 추적할 수 있도록 설계하는 것이 바람직하다.

**ItemController.java**
```java
@PostMapping("items/{itemId}/edit")
public String updateItem(@PathVariable("itemId") Long itemId, @ModelAttribute("form") BookForm form) {

    itemService.updateItem(itemId, form.getName(), form.getPrice(), form.getStockQuantity());

    return "redirect:/items";
}
```


**ItemService.java**
```java
@Transactional
    public void updateItem(Long itemId, String name, int price, int stockQuantity) {
        Item findItem = itemRepository.findOne(itemId);

        findItem.setName(name);
        findItem.setPrice(price);
        findItem.setStockQuantity(stockQuantity);
    }
```

즉, 위와 같은 설계가 바람직하다. 또한, 넘겨주는 파라미터가 많은 경우에는 Service 계층에 UpdateItemDto를 생성해 이를 넘겨주는 식으로 설계하는 것이 좋다.

컨트롤러에서 어설프게 엔티티를 생성하지 말고, 트랜잭션이 있는 계층에서 영속 상태로 조회하고 값을 변경해 변경 감지가 일어날 수 있도록 하자.

## 상품 주문

조회가 아닌 핵심 비즈니스 로직은 밖에서 엔티티를 찾아서 넘기는 것보다는, 식별자만 넘겨주고 내부에서 핵심 비즈니스 로직을 수행하도록 하는 것이 영속성 컨텍스트 내부에서 자연스럽게 변경 감지할 수 있어 권장되는 방식이다. 

## 주문 목록 검색/취소 

```java
@GetMapping(value = "/orders")
public String orderList(@ModelAttribute("orderSearch") OrderSearch orderSearch, Model model) {
    List<Order> orders = orderService.findOrders(orderSearch);
    model.addAttribute("orders", orders);
    return "order/orderList";
}
```

참고로 여기서 @ModelAttribute("orderSearch")를 해주면 orderSearch가 자동으로 model에 담긴다.

다음의 JPA 활용 2 강의에서는 API를 만들어보자!