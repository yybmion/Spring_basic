## 좋은 객체 지향 프로그래밍이란?
___


#### 역할과 구현을 분리
- 역할과 구현으로 구분하면 세상이 단순해지고, 유연해지며 변경도 편리해진다.
- 클라이언트는 대상의 역할(인터페이스)만 알면 된다.
- 클라이언트는 구현 개상의 **내부 구조를 몰라도** 된다.
- 클라니언트는 구현 대상의 **내부 구조가 변경**되어도 영향을 받지 않는다.
- 클라이언트는 구현 **대상 자체를 변경**해도 영향을 받지 않는다.

> 자바 언어의 다형성을 활용

- 역할 = 인터페이스
- 구현 = 인터페이스를 구현한 클래스, 구현 객체
- 둘을 명확히 분리할 필요가 있다.

Q.여기서 잠깐! 오버라이딩, 다형성이란?

```java
public class Parent {
    public static void main(String[] agrs) {

        Parent p1 = new Parent();
        Parent p2 = new Child();
        Parent p3 = new ChildOther();

        Parent[] arr = {p1, p2, p3};
        
        for(Parent item : arr) {
        	System.out.println("---------------");
        	if(item instanceof Child) {
        		System.out.println("is Child Type");
        	} 
        	if(item instanceof ChildOther) {
        		System.out.println("is ChildOther Type");
        	} 
        	if(item instanceof Parent) {
        		System.out.println("is Parent Type");
        	}
        }
    }
}
```

```
---------------
is Parent Type
---------------
is Child Type
is Parent Type
---------------
is ChildOther Type
is Parent Type
```

부모클래스를 제외한 자식클래스들은 두 가지의 클래스 타입을 가진것을 볼 수 있습니다.

이러한 것이 **다형성(ploymorphism)**입니다.

다형성의 예시는 매우 많습니다.

예를들어 **오버로딩(Overloading)**도 다형성을 구현하는 방법 중 하나입니다.

___

#### 다형성의 본질

**클라이언트를 변경하지 않고, 서버의 구현 기능을 유연하게 변경할 수 있다.**

**가장 중요한것은 인터페이스를 안정적으로 잘 설계하는 것이 중요하다.**