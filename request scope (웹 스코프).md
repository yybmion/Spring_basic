## 웹 스코프
___
> request : HTTP 요청 하나가 들어오고 나갈 떄 까지 유지되는 스코프,
각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고, 관리됩니다.

> 출처(https://ksr930.tistory.com/284)

**request 스코프로 설정 되있기 때문에 http 요청이 들어올때 스프링 컨테이너에 등록됩니다.**

그러니까 http 요청이 오지도 않아서 스프링 컨테이너에 등록이 안되어있으니 LogDemoController에서 
MyLogger를 의존성 주입이 가능할리가 없음.

> 오류


Error creating bean with name 'myLogger': Scope 'request' is not active for the
current thread; consider defining a scoped proxy for this bean if you intend to
refer to it from a singleton;

* 발생이유

스프링 어플리케이션을 실행하는 시점에 싱글톤 빈들은 생성해서 주입해준다.

하지만 request 스코프 빈은 아직생성되지 않았기 때문에 스프링 컨테이너에서 찾을 수 없다.

request 스코프 빈은 실제 고객의 요청이 와야 생성 및 초기화 할 수 있다. (HTTP requset 요청)

즉, Scope가 request이기 떄문에 실제 생성 및 초기화 시점은 HTTP requst 시점이 된다.

해결 방법

1. ObjectProvider을 이용한 방식

ObjectProvider클래스의 getObject() 메서드를 호출하는 시점에는 
http 요청이 활성화된 상태이므로 MyLogger 클래스의 request 스코프 빈 생성이 정상적으로 동작하는것입니다.

컨트롤러 서비스에서 외부 http 요청이 들어오면 MyLogger 객체를 생성하면서 MyLogger 클래스에서 @PostConstruct로 지정된 init() 메서드를 수행하게되고 uuid를 생성하게 됩니다. 
또한 LogDemoController, LogDemoService 클래스에서 각각 MyLogger 객체를 생성하지만 동일한 http 요청에 대해서는 동일한 스프링 빈 객체가 반환됩니다.

2. 프록시를 이용한 방법
 

컨트롤러 서비스에서 외부 http 요청이 들어오면 MyLogger 객체를 생성하면서 
MyLogger 클래스에서 @PostConstruct로 지정된 init() 메서드를 수행하게되고 uuid를 생성하게 됩니다. 
또한 LogDemoController, LogDemoService 클래스에서 각각 MyLogger 객체를 생성하지만 동일한 http 요청에 대해서는 동일한 스프링 빈 객체가 반환됩니다.
