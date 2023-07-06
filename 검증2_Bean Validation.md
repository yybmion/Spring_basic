# Bean Validation

- 지금까지의 검증은 매번 복잡한 코드를 작성해야한다는 단점이 있다.
- 따라서 다음과 같이 검증 로직을 표준화 시킬 수 있는데 이를 Bean Validation이라고 한다.

```java
public class Item {

 private Long id;

 @NotBlank
 private String itemName;

 @NotNull
 @Range(min = 1000, max = 1000000)
 private Integer price;

 @NotNull
 @Max(9999)
 private Integer quantity;
 //...
}
```

- Bean Validation을 사용하기 위해 build.gradle에 의존관계를 추가해준다.

```java
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

- 검증 애노테이션

@NotBlank : 빈값 + 공백만 있는 경우를 허용하지 않는다.
@NotNull : null 을 허용하지 않는다.
@Range(min = 1000, max = 1000000) : 범위 안의 값이어야 한다.
@Max(9999) : 최대 9999까지만 허용한다.

- 검증기 테스트

```java
import javax.validation.ValidatorFactory;
import java.util.Set;
public class BeanValidationTest {
    @Test
    void beanValidation() {
        ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
        Validator validator = factory.getValidator();
        Item item = new Item();
        item.setItemName(" "); //공백
        item.setPrice(0);
        item.setQuantity(10000);
        Set<ConstraintViolation<Item>> violations = validator.validate(item);
        for (ConstraintViolation<Item> violation : violations) {
            System.out.println("violation=" + violation);
            System.out.println("violation.message=" + violation.getMessage());
        }
    }
}
```

- 검증기 생성

```java
ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
Validator validator = factory.getValidator();
```

- 검증 실행

```java
Set<ConstraintViolation<Item>> violations = validator.validate(item);
```

검증을 실행하여 set에는 ConstrintViolation이라는 검증 오류가 담기는데 결과가 비어있으면 검증 오류가 없다는 뜻이다.

- 실행결과를 보면 위반사항 메시지가 보인다. 이 메시지는 자동으로 생성해주는 메시지고 개인이 따로 설정할 수 있다.

```java
violation={interpolatedMessage='공백일 수 없습니다', propertyPath=itemName, 
rootBeanClass=class hello.itemservice.domain.item.Item, 
messageTemplate='{javax.validation.constraints.NotBlank.message}'}
violation.message=공백일 수 없습니다

violation={interpolatedMessage='9999 이하여야 합니다', propertyPath=quantity, 
rootBeanClass=class hello.itemservice.domain.item.Item, 
messageTemplate='{javax.validation.constraints.Max.message}'}
violation.message=9999 이하여야 합니다

violation={interpolatedMessage='1000에서 1000000 사이여야 합니다', 
propertyPath=price, rootBeanClass=class hello.itemservice.domain.item.Item, 
messageTemplate='{org.hibernate.validator.constraints.Range.message}'}
violation.message=1000에서 1000000 사이여야 합니다
```

이러한 기능을 스프링은 이미 빈 검증기를 완전히 통합해두었다.

- 그렇다면 스프링 MVC는 어떻게 Bean Validator를 사용할까?

스프링 부트가 spring-boot-starter-validation 라이브러리를 넣으면 자동으로 Bean Validator를 인지하고 스프링에 통합한다.

- 스프링 부트는 자동으로 글로벌 Validator로 등록한다.

LocalValidatorFactoryBean 을 글로벌 Validator로 등록한다. 이 Validator는 @NotNull 같은 애노테이션을 보고 검증을 수행한다. 이렇게 글로벌 Validator가 적용되어 있기 때문에, @Valid , @Validated 만 적용하면 된다.
검증 오류가 발생하면, FieldError , ObjectError 를 생성해서 BindingResult 에 담아준다.

- 검증 순서
1. @ModelAttribute 각각의 필드에 타입 변환 시도
    1. 성공하면 다음으로
    2. 실패하면 typeMismatch 로 FieldError 추가
2. Validator 적용

따라서 바인딩이 성공한(타입 변환이 성공한) 필드만 Bean Validation 적용한다.

예를 들면)

- itemName 에 문자 "A" 입력 → 타입 변환 성공 → itemName 필드에 BeanValidation 적용
- price 에 문자 "A" 입력 → "A"를 숫자 타입 변환 시도 실패 → typeMismatch FieldError 추가 → price 필드는 BeanValidation 적용 X

- Bean Valdation 에러코드는 MessageCodesResolver를 통해 다양한 메시지 코드가 순서대로 생성된다.

@NotBlank
NotBlank.item.itemName
NotBlank.itemName
NotBlank.java.lang.String
NotBlank

@Range
Range.item.price
Range.price
Range.java.lang.Integer
Range

- 따라서 properties에서 메시지를 등록하여 다양한 메시지를 출력할 수 있다.

```java
#Bean Validation 추가
NotBlank={0} 공백X 
Range={0}, {2} ~ {1} 허용
Max={0}, 최대 {1}
```

만약 메시지가 properties에 없다면 애노데이션의 message 속성을 사용하고, 이것도 없다면 라이브러리가 제공하는 기본 값을 사용한다.

### 특정 필드가 아닌 오브젝트 오류

다음과 같이 @ScriptAssert()를 사용하면 된다.

```java
@Data
@ScriptAssert(lang = "javascript", script = "_this.price * _this.quantity >= 
10000")
public class Item {
 //...
}
```

하지만 이는 실제 사용해보면 제약이 많고 복잡하다. 따라서 글로벌 오류일 경우에는 @ScriptAssert를 사용하는 것 보다는 오브젝트 오류 곤련 부분만 직접 자바 코드로 작성하는 것이 좋다.

### Bean Validation - 수정에 적용

수정 기능도 추가 기능과 유사하다.

```java
@PostMapping("/{itemId}/edit")
    public String edit(@PathVariable Long itemId, @Validated @ModelAttribute Item
            item, BindingResult bindingResult) {

        //특정 필드 예외가 아닌 전체 예외
        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if (resultPrice < 10000) {
                bindingResult.reject("totalPriceMin", new Object[]{10000,
                        resultPrice}, null);
            }
        }
        if (bindingResult.hasErrors()) {
            log.info("errors={}", bindingResult);
            return "validation/v3/editForm";
        }
        itemRepository.update(itemId, item);
        return "redirect:/validation/v3/items/{itemId}";
    }
```

### Bean Validation -한계

- 등록시에는 수량을 최대 9999개라는 제약을 두었지만, 수정시에는 수량을 무제한으로 변경할 수 있게 하려고한다.
- 이때 검증 조건의 충돌이 발생하고, 등록과 수정은 같은 BeanValidation
을 적용할 수 없다. 이 문제를 어떻게 해결할 수 있을까?

### Bean Validation - groups

방법 2가지

- BeanValidation의 groups 기능을 사용한다.
- Item을 직접 사용하지 않고, ItemSaveForm, ItemUpdateForm 같은 폼 전송을 위한 별도의 모델 객체를 만들어서 사용한다.

먼저 Groups기능을 살펴보자. Groups는 등록시에 검증할 기능과 수정시에 검증할 기능을 각각 그룹으로 나누어 적용할 수 있다.

- 수정용 groups

```java
package hello.itemservice.domain.item;
public interface UpdateCheck {
}
```

- 저장용 groups

```java
package hello.itemservice.domain.item;
public interface SaveCheck {
}
```

따로 만들어주고

- Item - groups 적용

```java
@Data
    public class Item {
        @NotNull(groups = UpdateCheck.class) //수정시에만 적용
        private Long id;
        @NotBlank(groups = {SaveCheck.class, UpdateCheck.class})
        private String itemName;
        @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
        @Range(min = 1000, max = 1000000, groups = {SaveCheck.class,
                UpdateCheck.class})
        private Integer price;
        @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
        @Max(value = 9999, groups = SaveCheck.class) //등록시에만 적용
        private Integer quantity;
        public Item() {
        }
        public Item(String itemName, Integer price, Integer quantity) {
            this.itemName = itemName;
            this.price = price;
            this.quantity = quantity;
        }
    }
```

각 필드마다 groups를 통해 검증을 달리 만들 수 있다.

- 컨트롤러

```java
@PostMapping("/add")
public String addItemV2(@Validated(SaveCheck.class) @ModelAttribute Item item,
BindingResult bindingResult, RedirectAttributes redirectAttributes) {
 //...
}
```

@Validated(SaveCheck.class)를 적용

groups 기능을 사용해서 등록과 수정시에 각각 다르게 검증을 할 수 있었다. 그런데 groups 기능을 사용하니 Item 은 물론이고, 전반적으로 복잡도가 올라갔다.
사실 groups 기능은 실제 잘 사용되지는 않는데, 그 이유는 실무에서는 주로 다음에 등장하는 등록용 폼 객체와 수정용 폼 객체를 분리해서 사용하기 때문이다.

### From 전송 객체 분리 - 소개

<aside>
💡

실무에서는 회원 등록시 회원과 관련된 데이터만 전달받는 것이 아니라, 약관 정보도 추가로 받는 등 Item 과 관계없는 수 많은 부가 데이터가 넘어온다. 그래서 보통 Item 을 직접 전달받는 것이 아니라, 복잡한 폼의 데이터를 컨트롤러까지 전달할 별도의 객체를 만들어서 전달한다. 예를 들면 ItemSaveForm 이라는 폼을 전달받는 전용 객체를 만들어서
@ModelAttribute 로 사용한다. 이것을 통해 컨트롤러에서 폼 데이터를 전달 받고, 이후 컨트롤러에서 필요한 데이터를 사용해서 Item 을 생성한다.

</aside>

- HTML Form -> ItemSaveForm -> Controller -> Item 생성 -> Repository
    - 장점: 전송하는 폼 데이터가 복잡해도 거기에 맞춘 별도의 폼 객체를 사용해서 데이터를 전달 받을 수
    있다. 보통 등록과, 수정용으로 별도의 폼 객체를 만들기 때문에 검증이 중복되지 않는다.
    - 단점: 폼 데이터를 기반으로 컨트롤러에서 Item 객체를 생성하는 변환 과정이 추가된다.
- 폼 데이터 전달을 위한 별도의 객체를 사용하고, 등록, 수정용 폼 객체를 나누면 등록, 수정이 완전히 분리되기 때문에 groups 를 적용할 일은 드물다

이제 본격적으로 객체를 분리해보자.

### Form 전송 객체 분리 - 개발

groups를 사용하지 않기 때문에 Item클래스의 검증 코드를 제거해도 된다.

```java
@Data
public class Item {
 private Long id;
 private String itemName;
 private Integer price;
 private Integer quantity;
}
```

대신 ItemSaveForm에 Item에 대한 검증기를 생성한다.

```java
@Data
public class ItemSaveForm {
 @NotBlank
 private String itemName;
 @NotNull
 @Range(min = 1000, max = 1000000)
 private Integer price;
 @NotNull
 @Max(value = 9999)
 private Integer quantity;
}
```

ItemUpdateForm

```java
@Data
public class ItemUpdateForm {
 @NotNull
 private Long id;
 @NotBlank
 private String itemName;
 @NotNull
 @Range(min = 1000, max = 1000000)
 private Integer price;
 //수정에서는 수량은 자유롭게 변경할 수 있다.
 private Integer quantity;
}
```

컨트롤러를 통해 알아보자.

```java
@PostMapping("/add")
 public String addItem(@Validated @ModelAttribute("item") ItemSaveForm form,
BindingResult bindingResult, RedirectAttributes redirectAttributes) {
 //특정 필드 예외가 아닌 전체 예외
 if (form.getPrice() != null && form.getQuantity() != null) {
 int resultPrice = form.getPrice() * form.getQuantity();
 if (resultPrice < 10000) {
 bindingResult.reject("totalPriceMin", new Object[]{10000,
resultPrice}, null);
 }
 }
 if (bindingResult.hasErrors()) {
 log.info("errors={}", bindingResult);
 return "validation/v4/addForm";
 }
 //성공 로직
 Item item = new Item();
 item.setItemName(form.getItemName());
 item.setPrice(form.getPrice());
 item.setQuantity(form.getQuantity());
 Item savedItem = itemRepository.save(item);
 redirectAttributes.addAttribute("itemId", savedItem.getId());
 redirectAttributes.addAttribute("status", true);
 return "redirect:/validation/v4/items/{itemId}";
 }
 @GetMapping("/{itemId}/edit")
 public String editForm(@PathVariable Long itemId, Model model) {
 Item item = itemRepository.findById(itemId);
 model.addAttribute("item", item);
 return "validation/v4/editForm";
 }
```

ItemSaveForm 부분만 보자.

- 폼 객체 바인딩

```java
@PostMapping("/add")
public String addItem(@Validated @ModelAttribute("item") ItemSaveForm form,
BindingResult bindingResult, RedirectAttributes redirectAttributes) {
 //...
}
```

전에 코드에서는 Item 객체를 받았지만 지금은 ItemSaveForm을 전달 받는다. @Validated 검증도 수행하고, BindingResult로 검증 결과도 받는다.

주의! @ModelAttribute(”item”) 이라고 나타내주어야하는데, 그 이유는 원래 @ModelAttribute에 아무것도 적어주지 않으면 ItemSaveForm이름으로 MVC MODEL에 담기게 된다. 그렇게 되면 TH:OBJECT 이름도 변경해주어야하는데 그 수고를 덜기 위해 ITEM으로 지정해준다.

위에서 본 단점으로 뽑은 폼객체를 Item으로 변환해야한다는 번거로움이 있다.

```java
//성공 로직
Item item = new Item();
item.setItemName(form.getItemName());
item.setPrice(form.getPrice());
item.setQuantity(form.getQuantity());
Item savedItem = itemRepository.save(item);
```

Form 전송 객체 분리해서 등록과 수정에 딱 맞는 기능을 구성하고, 검증도 명확히 분리했다.

### Bean validation - http 메시지 컨버터

> 참고
@ModelAttribute 는 HTTP 요청 파라미터(URL 쿼리 스트링, POST Form)를 다룰 때 사용한다.
@RequestBody 는 HTTP Body의 데이터를 객체로 변환할 때 사용한다. 주로 API JSON 요청을 다룰 때 사용한다.
> 

```java
@Slf4j
@RestController
@RequestMapping("/validation/api/items")
public class ValidationItemApiController {
 @PostMapping("/add")
 public Object addItem(@RequestBody @Validated ItemSaveForm form,
 BindingResult bindingResult) {
 log.info("API 컨트롤러 호출");
 if (bindingResult.hasErrors()) {
 log.info("검증 오류 발생 errors={}", bindingResult);
 return bindingResult.getAllErrors();
 }
 log.info("성공 로직 실행");
 return form;
 }
}
```

API의 경우 3가지 경우를 나누어 생각해야 한다.
1. 성공 요청: 성공
2. 실패 요청: JSON을 객체로 생성하는 것 자체가 실패함
3. 검증 오류 요청: JSON을 객체로 생성하는 것은 성공했고, 검증에서 실패함

- 실패 요청

```java
POST http://localhost:8080/validation/api/items/add
{"itemName":"hello", "price":"A", "quantity": 10}
```

- 실패 요청 결과

```json
{
 "timestamp": "2021-04-20T00:00:00.000+00:00",
 "status": 400,
 "error": "Bad Request",
 "message": "",
 "path": "/validation/api/items/add"
}
```

HttpMessageConverter 에서 요청 JSON을 ItemSaveForm 객체로 생성하는데 실패한다.
이 경우는 ItemSaveForm 객체를 만들지 못하기 때문에 컨트롤러 자체가 호출되지 않고 그 전에 예외가 발생한다. 물론 Validator도 실행되지 않는다.

- 검증 오류 요청

바인딩은 성공하지만 검증에서 오류가 발생하는 경우

```json
POST http://localhost:8080/validation/api/items/add
{"itemName":"hello", "price":1000, "quantity": 10000}
```

```json
[
 {
 "codes": [
 "Max.itemSaveForm.quantity",
 "Max.quantity",
 "Max.java.lang.Integer",
 "Max"
 ],
 "arguments": [
 {
 "codes": [
 "itemSaveForm.quantity",
 "quantity"
 ],
 "arguments": null,
 "defaultMessage": "quantity",
 "code": "quantity"
 },
 9999
 ],
 "defaultMessage": "9999 이하여야 합니다",
 "objectName": "itemSaveForm",
 "field": "quantity",
 "rejectedValue": 10000,
 "bindingFailure": false,
 "code": "Max"
 }
]
```

return bindingResult.getAllErrors(); 는 ObjectError 와 FieldError 를 반환한다. 스프링이 이 객체를 JSON으로 변환해서 클라이언트에 전달했다.

<aside>
💡

**@ModelAttribute vs @RequestBody**
HTTP 요청 파리미터를 처리하는 @ModelAttribute 는 각각의 필드 단위로 세밀하게 적용된다. 그래서 특정 필드에 타입이 맞지 않는 오류가 발생해도 나머지 필드는 정상 처리할 수 있었다.
HttpMessageConverter 는 @ModelAttribute 와 다르게 각각의 필드 단위로 적용되는 것이 아니라, 전체 객체 단위로 적용된다.
따라서 메시지 컨버터의 작동이 성공해서 ItemSaveForm 객체를 만들어야 @Valid , @Validated 가 적용된다.

- @ModelAttribute 는 필드 단위로 정교하게 바인딩이 적용된다. 특정 필드가 바인딩 되지 않아도 나머지 필드는 정상 바인딩 되고, Validator를 사용한 검증도 적용할 수 있다.
- @RequestBody 는 HttpMessageConverter 단계에서 JSON 데이터를 객체로 변경하지 못하면 이후 단계 자체가 진행되지 않고 예외가 발생한다. 컨트롤러도 호출되지 않고, Validator도 적용할 수 없다.

</aside>
