## MVC
___
하나의 서블릿이나 JSP만으로 비즈니스 로직과 뷰 렌더링까지 모두 처리하게 되면, 너무 많은 역할을
하게되고, 결과적으로 유지보수가 어려워진다.

> MVC 패턴은 지금까지 학습한 것 처럼 하나의 서블릿이나, JSP로 처리하던 것을 컨트롤러(Controller)와
뷰(View)라는 영역으로 서로 역할을 나눈 것을 말한다.

**컨트롤러 - 모델 - 뷰**

- 컨트롤러 : : HTTP 요청을 받아서 파라미터를 검증하고, 비즈니스 로직을 실행한다. 그리고 뷰에 전달할 결과
데이터를 조회해서 모델에 담는다.
- 모델: 뷰에 출력할 데이터를 담아둔다. 뷰가 필요한 데이터를 모두 모델에 담아서 전달해주는 덕분에 뷰는
비즈니스 로직이나 데이터 접근을 몰라도 되고, 화면을 렌더링 하는 일에 집중할 수 있다.
- 뷰: 모델에 담겨있는 데이터를 사용해서 화면을 그리는 일에 집중한다. 여기서는 HTML을 생성하는 부분을
말한다.

![70](https://user-images.githubusercontent.com/113106136/228417586-1195854b-b93e-476d-b0ab-a7923c67e448.png)

___
MVC 패턴 적용

> 참고자료 3장 21

회원등록 / 회원저장 / 회원목록

각각의 컨트롤러와 뷰를 만들어준다.

___

회원등록에서의 뷰는 상경로를 사용하였는데 상대경로를 사용하면 폼 전송시 현재 URL이 속한 계층 경로 + save가 호출된다.

현재 계층 경로: /servlet-mvc/members/
결과: /servlet-mvc/members/save

___

회원저장에서는 request.setAttribute로 모델에 데이터를 보관하여 나중에 뷰를 통해 화면에 나타낼 데이터를 저장한다.

뷰는 request.getAttribute() 를 사용해서 데이터를 꺼내면 된다.

___

회원목록조회는 List를 통해 회원목록을 저장하여 똑같이 모델에 데이터를 저장한다.

___

MVC패턴의 한계

컨트롤러는 딱 봐도 중복이 많고, 필요하지 않는 코드들도 많이 보인다.

> 포워드 중복
```java
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response);
```

> vIewPath 중복
```java
String viewPath = "/WEB-INF/views/new-form.jsp";
```

> 사용하지 않는 코드
```java
HttpServletRequest request, HttpServletResponse response
```

___

## FrontController??
- 프론트 컨트롤러 서블릿 하나로 클라이언트의 요청을 받음
- 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출
- 입구를 하나로!
- 공통 처리 가능
- 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 됨

### ControllerV1

서블릿과 비슷한 모양의 컨트롤러 인터페이스 도입. 각 컨트롤러는 이 인터페이스를 구현하면됨.

> 참고자료 4장 3쪽 

내부로직은 기존 컨트롤러와 거의 같음

공통으로 처리할 프론트 컨트롤러를 보자.

먼저 눈에 띄는것은 생성자에 Map (key,value)형태로 나타내어 해당 주소가 오면 특정 컨트롤러를 수행하게 만들어졌다.

urlPatterns = "/front-controller/v1/*" : /front-controller/v1 를 포함한 하위 모든 요청은
이 서블릿에서 받아들인다.

___

### ControllerV2 (View의 분리)

**중복**
```java
String viewPath = "/WEB-INF/views/new-form.jsp";
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response);
```
MyView 생성!
render메소드를 만들어 여기에 위에 해당하는 뷰 중복 로직을 한곳에 모았다.

> 참고자료 4장 10

이번 컨트롤러도 V1과 크게 다를게 없지만 MyView에서 중복부분을 처리하는 부분만 달라졌다.

```java
public class MemberFormControllerV2 implements ControllerV2 {
 @Override
 public MyView process(HttpServletRequest request, HttpServletResponse 
response) throws ServletException, IOException {
 return new MyView("/WEB-INF/views/new-form.jsp");
 }
}
```
V2프론트컨트롤러
```java
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse 
response)
      throws ServletException, IOException {
    String requestURI = request.getRequestURI();
    
    ControllerV2 controller = controllerMap.get(requestURI);
    if (controller == null) {
        response.setStatus(HttpServletResponse.SC_NOT_FOUND);
        return;
  }
      MyView view = controller.process(request, response);
      view.render(request, response);
    }
}
```
- ControllerV2의 반환 타입이 MyView 이므로 프론트 컨트롤러는 컨트롤러의 호출 결과로 MyView 를 반환
받는다. 그리고 view.render() 를 호출하면 forward 로직을 수행해서 JSP가 실행된다.

- 프론트 컨트롤러의 도입으로 MyView 객체의 render() 를 호출하는 부분을 모두 일관되게 처리할 수
있다. 각각의 컨트롤러는 MyView 객체를 생성만 해서 반환하면 된다.



