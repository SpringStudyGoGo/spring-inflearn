# 리터럴 대체 - |...|
`|...|` :이렇게 사용한다.
타임리프에서 문자와 표현식 등은 분리되어 있기 때문에 더해서 사용해야 한다.
```html
<span th:text="'Welcome to our application, ' + ${user.name} + '!'">
 ```
다음과 같이 리터럴 대체 문법을 사용하면, 더하기 없이 편리하게 사용할 수 있다. 
```html
<span th:text="|Welcome to our application, ${user.name}!|">
```
결과를 다음과 같이 만들어야 하는데
```html
location.href='/basic/items/add'
```
그냥 사용하면 문자와 표현식을 각각 따로 더해서 사용해야 하므로 다음과 같이 복잡해진다.
```html
th:onclick="'location.href=' + '\'' + @{/basic/items/add} + '\''"
```
리터럴 대체 문법을 사용하면 다음과 같이 편리하게 사용할 수 있다. 
```html
th:onclick="|location.href='@{/basic/items/add}'|"
```

---
# 속성 변경 - th:action 
## th:action
1. HTML form에서 `action` 에 값이 없으면 현재 URL에 데이터를 전송한다.
2. 상품 등록 폼의 URL과 실제 상품 등록을 처리하는 URL을 똑같이 맞추고 HTTP 메서드로 두 기능을 구분한다.
   - 상품 등록 폼: GET `/basic/items/add`
   - 상품 등록 처리: POST `/basic/items/add`

3. 이렇게 하면 하나의 URL로 등록 폼과, 등록 처리를 깔끔하게 처리할 수 있다.

---
# 상품 등록 처리 - @ModelAttribute
이제 상품 등록 폼에서 전달된 데이터로 실제 상품을 등록 처리해보자. 상품 등록 폼은 다음 방식으로 서버에 데이터를 전달한다.
## POST - HTML Form
	content-type: application/x-www-form-urlencoded
메시지 바디에 쿼리 파리미터 형식으로 전달

	itemName=itemA&price=10000&quantity=10 
예) 회원 가입, 상품 주문, HTML Form 사용
요청 파라미터 형식을 처리해야 하므로 `@RequestParam` 을 사용하자 

## 상품 등록 처리 - @RequestParam
  
```java
	@PostMapping("/add")
    public String addItemV1(@RequestParam String itemName,
                       @RequestParam int price,
                       @RequestParam Integer quantity,
                       Model model) {
        Item item = new Item();
        item.setItemName(itemName);
        item.setPrice(price);
        item.setQuantity(quantity);

        itemRepository.save(item);

        model.addAttribute("item", item);
        return "basic/addForm";
    }

```
먼저 `@RequestParam String itemName` : `itemName` 요청 파라미터 데이터를 해당 변수에 받는다. 
`Item` 객체를 생성하고 `itemRepository` 를 통해서 저장한다.
저장된 `item` 을 모델에 담아서 뷰에 전달한다.
중요: 여기서는 상품 상세에서 사용한 `item.html` 뷰 템플릿을 그대로 재활용한다. 실행해서 상품이 잘 저장되는지 확인하자.

## 상품 등록 처리 - @ModelAttribute
`@RequestParam` 으로 변수를 하나하나 받아서 `Item` 을 생성하는 과정은 불편했다.
이번에는 `@ModelAttribute` 를 사용해서 한번에 처리해보자.

```java
    @PostMapping("/add")
    public String addItemV2(@ModelAttribute("item") Item item) {
        itemRepository.save(item);
        return "basic/addForm";
    }  
```
### `@ModelAttribute` - 요청 파라미터 처리
`@ModelAttribute` 는 `Item` 객체를 생성하고, 요청 파라미터의 값을 프로퍼티 접근법(setXxx)으로 입력해준다.
### @ModelAttribute - Model 추가
`@ModelAttribute` 는 중요한 한가지 기능이 더 있는데, 
바로 모델(Model)에 `@ModelAttribute` 로 지정한 객체를 자동으로 넣어준다. 
모델에 데이터를 담을 때는 이름이 필요하다. 
이름은 `@ModelAttribute` 에 지정한 `name(value)` 속성을 사용한다. 만약 다음과 같이 `@ModelAttribute` 의 이름을 다르게 지정하면 다른 이름으로 모델에 포함된다.

- `@ModelAttribute("hello") Item` → `item` 이름을 `hello` 로 지정 
- `model.addAttribute("hello", item);` 모델에 `hello` 이름으로 저장

## addItemV3 - 상품 등록 처리 - ModelAttribute 이름 생략

```java
@PostMapping("/add")
    public String addItemV3(@ModelAttribute Item item) {
        itemRepository.save(item);
        return "basic/addForm";
    }
```

`@ModelAttribute` 의 이름을 생략할 수 있다.
### 주의
`@ModelAttribute` 의 이름을 생략하면 모델에 저장될 때 클래스명을 사용한다. 이때 클래스의 첫글자만 소문자로 변경해서 등록한다.

## addItemV4 - 상품 등록 처리 - ModelAttribute 전체 생략

```java
    @PostMapping("/add")
    public String addItemV4(Item item) {
        itemRepository.save(item);
        return "basic/addForm";
    }
```

---
# PRG Post/Redirect/Get
문제 발생 → 상품 등록을 완료하고 웹 브라우저의 새로고침 버튼을 클릭해보자.
상품이 계속해서 **중복 등록**되는 것을 확인할 수 있다.

## 전체 흐름
![](https://velog.velcdn.com/images/mini_mouse_/post/52723a82-1c05-4b16-bcba-e2a675378c69/image.png)

## POST 등록 후 새로 고침
![](https://velog.velcdn.com/images/mini_mouse_/post/ccebf3f8-9090-4374-b0e0-b04485ca6c1b/image.png)

## 해결방법: POST, Redirect GET
![](https://velog.velcdn.com/images/mini_mouse_/post/e621e75a-0fa4-4a4b-b9b9-bcb7bb6672ba/image.png)

새로 고침 문제를 해결하려면 상품 저장 후에 뷰 템플릿으로 이동하는 것이 아니라, 상품 상세 화면으로 **리다이렉트**를 호출해주면 된다.

---
# RedirectAttributes
```java
@PostMapping("/add")
    public String addItemV6(Item item, 
    RedirectAttributes redirectAttributes) {
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/basic/items/{itemId}";
}
```

## RedirectAttributes
`RedirectAttributes` 를 사용하면 URL 인코딩도 해주고, 
`pathVarible` , 쿼리 파라미터까지 처리해준다.

`redirect:/basic/items/{itemId}`
- **pathVariable** 바인딩: `{itemId}`
- **나머지는 쿼리 파라미터**로 처리: `?status=true`

## ${param.status}
>타임리프에서 쿼리 파라미터를 편리하게 조회하는 기능

원래는 컨트롤러에서 모델에 직접 담고 값을 꺼내야 한다. 그런데 쿼리 파라미터는 자주 사용해서 타임리프에서 직접 지원한다.

---
