## AppConfig 리펙터링
___
현재 AppConfig를 보면 중복이 있고, 역할에 따른 구현이 잘 안보인다.

```java
public class AppConfig {
 public MemberService memberService() {
    return new MemberServiceImpl(memberRepository());
 }
 public OrderService orderService() {
    return new OrderServiceImpl(
      memberRepository(),
      discountPolicy());
 }
 public MemberRepository memberRepository() {
    return new MemoryMemberRepository();
 }
 public DiscountPolicy discountPolicy() {
    return new FixDiscountPolicy();
 }
}
```

new MemoryMemberRepository() 이 부분이 중복 제거되었다. 이제 MemoryMemberRepository 를
다른 구현체로 변경할 때 한 부분만 변경하면 된다.
AppConfig 를 보면 역할과 구현 클래스가 한눈에 들어온다. 애플리케이션 전체 구성이 어떻게 되어있는지
빠르게 파악할 수 있다.

___

이제 할인정책을 정률 할인 정책으로 바꾸어보자

어느 부분을 바꾸어야 하겠는가?

Appconfig 부분만 변경하면 된다!!
![29](https://user-images.githubusercontent.com/113106136/214109062-855bb765-ae1f-4690-bb68-c539d702413c.png)
![30](https://user-images.githubusercontent.com/113106136/214109148-657d2201-d693-4588-9c76-31edc1057583.png)

AppConfig 에서 할인 정책 역할을 담당하는 구현을 FixDiscountPolicy RateDiscountPolicy
객체로 변경했다.
이제 할인 정책을 변경해도, 애플리케이션의 구성 역할을 담당하는 AppConfig만 변경하면 된다. 
클라이언트 코드인 OrderServiceImpl 를 포함해서 사용 영역의 어떤 코드도 변경할 필요가 없다.
구성 영역은 당연히 변경된다. 구성 역할을 담당하는 AppConfig를 애플리케이션이라는 공연의 기획자로
생각하자. 공연 기획자는 공연 참여자인 구현 객체들을 모두 알아야 한다.

___

좋은 객체 지향 설계의 5가지 원칙 적용

여기서 3가지 SRP, DIP, OCP 적용
SRP 단일 책임 원칙
한 클래스는 하나의 책임만 가져야 한다.

- 클라이언트 객체는 직접 구현 객체를 생성하고, 연결하고, 실행하는 다양한 책임을 가지고 있음
- SRP 단일 책임 원칙을 따르면서 관심사를 분리함
- 구현 객체를 생성하고 연결하는 책임은 AppConfig가 담당
- 클라이언트 객체는 실행하는 책임만 담당


DIP 의존관계 역전 원칙
프로그래머는 “추상화에 의존해야지, 구체화에 의존하면 안된다.” 의존성 주입은 이 원칙을 따르는 방법 중
하나다.

- 새로운 할인 정책을 개발하고, 적용하려고 하니 클라이언트 코드도 함께 변경해야 했다. 왜냐하면 기존
클라이언트 코드( OrderServiceImpl )는 DIP를 지키며 DiscountPolicy 추상화 인터페이스에
의존하는 것 같았지만, FixDiscountPolicy 구체화 구현 클래스에도 함께 의존했다.
- 클라이언트 코드가 DiscountPolicy 추상화 인터페이스에만 의존하도록 코드를 변경했다.
- 하지만 클라이언트 코드는 인터페이스만으로는 아무것도 실행할 수 없다.
- AppConfig가 FixDiscountPolicy 객체 인스턴스를 클라이언트 코드 대신 생성해서 클라이언트
코드에 의존관계를 주입했다. 이렇게해서 DIP 원칙을 따르면서 문제도 해결했다.


OCP
소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀 있어야 한다

- 다형성 사용하고 클라이언트가 DIP를 지킴
- 애플리케이션을 사용 영역과 구성 영역으로 나눔
- AppConfig가 의존관계를 FixDiscountPolicy RateDiscountPolicy 로 변경해서 클라이언트
코드에 주입하므로 클라이언트 코드는 변경하지 않아도 됨
- 소프트웨어 요소를 새롭게 확장해도 사용 영역의 변경은 닫혀 있다!

