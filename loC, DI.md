## loC, DI, 컨테이너
___

제어의 역전 IoC(Inversion of Control)

- 기존 프로그램은 클라이언트 구현 객체가 스스로 필요한 서버 구현 객체를 생성하고, 연결하고, 실행했다. 
한마디로 구현 객체가 프로그램의 제어 흐름을 스스로 조종했다. 개발자 입장에서는 자연스러운 흐름이다.
- 반면에 AppConfig가 등장한 이후에 구현 객체는 자신의 로직을 실행하는 역할만 담당한다. 프로그램의
제어 흐름은 이제 AppConfig가 가져간다. 예를 들어서 OrderServiceImpl 은 필요한 인터페이스들을
호출하지만 어떤 구현 객체들이 실행될지 모른다. 
- 프로그램에 대한 제어 흐름에 대한 권한은 모두 AppConfig가 가지고 있다. 심지어 OrderServiceImpl
도 AppConfig가 생성한다. 그리고 AppConfig는 OrderServiceImpl 이 아닌 OrderService 
인터페이스의 다른 구현 객체를 생성하고 실행할 수 도 있다. 그런 사실도 모른체 OrderServiceImpl 은
묵묵히 자신의 로직을 실행할 뿐이다.
- 이렇듯 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것을 제어의 역전(IoC)이라
한다.

IoC 컨테이너, DI 컨테이너

AppConfig 처럼 객체를 생성하고 관리하면서 의존관계를 연결해 주는 것을
IoC 컨테이너 또는 DI 컨테이너라 한다. 
의존관계 주입에 초점을 맞추어 최근에는 주로 DI 컨테이너라 한다.
또는 어샘블러, 오브젝트 팩토리 등으로 불리기도 한다.

___

스프링으로 전환

```java

@Configuration
public class AppConfig {
    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
 }
    @Bean
    public OrderService orderService() {
      return new OrderServiceImpl(
          memberRepository(),
           discountPolicy());
 }
    @Bean
    public MemberRepository memberRepository() {
      return new MemoryMemberRepository();
 }
    @Bean
    public DiscountPolicy discountPolicy() {
      return new RateDiscountPolicy();
 }
}
```
AppConfig에 설정을 구성한다는 뜻의 @Configuration 을 붙여준다.
각 메서드에 @Bean 을 붙여준다. 이렇게 하면 스프링 컨테이너에 스프링 빈으로 등록한다.

적용하기
```java
public class MemberApp {
  public static void main(String[] args) {
// AppConfig appConfig = new AppConfig();
// MemberService memberService = appConfig.memberService();
       ApplicationContext applicationContext = new
  AnnotationConfigApplicationContext(AppConfig.class);
       MemberService memberService =
applicationContext.getBean("memberService", MemberService.class);

       Member member = new Member(1L, "memberA", Grade.VIP);
       memberService.join(member);
 
        Member findMember = memberService.findMember(1L);
        System.out.println("new member = " + member.getName());
        System.out.println("find Member = " + findMember.getName());
 }
}
```
스프링 컨테이너
- ApplicationContext 를 스프링 컨테이너라 한다.
- 기존에는 개발자가 AppConfig 를 사용해서 직접 객체를 생성하고 DI를 했지만, 이제부터는 스프링
컨테이너를 통해서 사용한다.
- 스프링 컨테이너는 @Configuration 이 붙은 AppConfig 를 설정(구성) 정보로 사용한다. 여기서 @Bean
이라 적힌 메서드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록한다. 이렇게 스프링 컨테이너에
등록된 객체를 스프링 빈이라 한다.
- 스프링 빈은 @Bean 이 붙은 메서드의 명을 스프링 빈의 이름으로 사용한다. ( memberService ,
orderService )
- 이전에는 개발자가 필요한 객체를 AppConfig 를 사용해서 직접 조회했지만, 이제부터는 스프링
컨테이너를 통해서 필요한 스프링 빈(객체)를 찾아야 한다. 스프링 빈은
applicationContext.getBean() 메서드를 사용해서 찾을 수 있다.
- 기존에는 개발자가 직접 자바코드로 모든 것을 했다면 이제부터는 스프링 컨테이너에 객체를 스프링 빈으로
등록하고, 스프링 컨테이너에서 스프링 빈을 찾아서 사용하도록 변경되었다.
