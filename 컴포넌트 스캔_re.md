@Bean 하는게 수동 빈 등록
@ComponentScan 을 통해서 @Component 어노테이션을 빈 등록 하는것이 자동 빈 등록

생성자에 @Autowired 를 지정하면, 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입한다.
이때 기본 조회 전략은 타입이 같은 빈을 찾아서 주입한다.
getBean(MemberRepository.class) 와 동일하다고 이해하면 된다.
더 자세한 내용은 뒤에서 설명한다.

![image](https://github.com/yybmion/Spring_basic/assets/113106136/e6b4cb57-a418-485f-a4e4-66d8ee3d9083)

___
```java
@Component
public class FixDiscountPolicy implements DiscountPolicy {}
```

```java
@Component
public class RateDiscountPolicy implements DiscountPolicy {}
```

```java
@Autowired
private DiscountPolicy discountPolicy
```

하면  fixdiscountpolicy를 주입받아야하는지 rateDiscountPolicy를 주입받아야하는지 모른다.
NoUniqueBeanDefinitionException 오류가 발생한다.
오류메시지가 친절하게도 하나의 빈을 기대했는데 fixDiscountPolicy , rateDiscountPolicy 2개가 발견되었다고 알려준다.

그래서 어떻게 하냐>
@Autowired 는 타입 매칭을 시도하고, 이때 여러 빈이 있으면 필드 이름, 파라미터 이름으로 빈 이름을 추가 매칭한다.

@Autowired
private DiscountPolicy discountPolicy 이걸

@Autowired
private DiscountPolicy rateDiscountPolicy 로 변경

필드 명이 rateDiscountPolicy 이므로 정상 주입된다.
필드 명 매칭은 먼저 타입 매칭을 시도 하고 그 결과에 여러 빈이 있을 때 추가로 동작하는 기능이다.

조회한 빈이 모두 필요할 때, List, Map

```java
public DiscountService(Map<String, DiscountPolicy> policyMap,
List<DiscountPolicy> policies) {
this.policyMap = policyMap;
this.policies = policies;
System.out.println("policyMap = " + policyMap);
System.out.println("policies = " + policies);
}
```

이거 보면 그냥 자동으로 DiscountService는 Map으로 모든 DiscountPolicy 를 주입받는다. 이때 fixDiscountPolicy ,
rateDiscountPolicy 가 주입된다.

new AnnotationConfigApplicationContext() 를 통해 스프링 컨테이너를 생성한다.
AutoAppConfig.class , DiscountService.class 를 파라미터로 넘기면서 해당 클래스를 자동으로
스프링 빈으로 등록한다.

편리한 자동 기능을 기본으로 사용하자
직접 등록하는 기술 지원 객체는 수동 등록
다형성을 적극 활용하는 비즈니스 로직은 수동 등록을 고민해보자
