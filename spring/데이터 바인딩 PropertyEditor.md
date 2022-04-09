# [스프링 프레임워크 핵심기술] - 데이터 바인딩 PropertyEditor

---

> 백기선님의 **스프링 프레임워크 핵심 기술**이라는 강좌를 들으며 공부한 내용을 정리한 글입니다.
>

DataBinder라는 Spring의 핵심 인터페이스를 알아본다.

# 1. 데이터 바인딩

데이터 바인딩이란 기술적인 관점에서 봤을 때 프로터피 값을 어떤 타겟 객체에 설정하는 기능이라고 정의 할 수 있다. 즉, 사용자가 어떤 Application 도메인 객체에 값을 동적으로 할당하고 싶을 때

사용한다.

예를들면, 사용자가 주로 입력하는 값은 문자열 값인데 객체가 가지고 있는 다양한 타입(int, long, boolean, Date 등)으로 인식해야 하는 경우가 있다. 이처럼 객체가 가지고 있는 다양한 타입을 변환해주는 기능을 데이터 바인딩이라고 한다.

---

# 2. 예제

먼저 가장 고전적인 방법으로 데이터 바인딩에 대해 알아보자.

간단한 테스트를 위해 Event라는 도메인 클레스를 만들어준다. 가장 기본적인 Java Beans이다.

```java
public class Event {
    private Integer id;

    private String title;

    public Event(Integer id) {
        this.id = id;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    @Override
    public String toString() {
        return "Event{" +
                "id=" + id +
                ", title='" + title + '\'' +
                '}';
    }
}
```

그리고 Controller도 만들어준다.

```java
@RestController
public class EventController {

    @GetMapping("/event/{event}")
    public String getEvent(@PathVariable Event event) {
        System.out.println(event);
        return event.getId().toString();
    }
}
```

Mapping되는 **{event}**에 id값을 입력받는데, 입력받는 숫자를 Event 타입으로 변환해서 받아야한다.

현재 상태에서는 Event 타입으로 변환할 수 없다.

Test 코드를 작성해서 확인해보자.

```java
@RunWith(SpringRunner.class)
@WebMvcTest
class EventControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void getTest() throws Exception {

        mockMvc.perform(get("/event/1")) // 이런 요청을 보냈을 때
                .andExpect(status().isOk()) // 응답이 200으로 나오고,
                .andExpect(content().string("1"));  // 결과는 "1"이 나와야 한다.
    }
}
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e3876424-891c-450d-88a6-bb5d5ac764ff/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e3876424-891c-450d-88a6-bb5d5ac764ff/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8d274cdd-bf86-457d-a83c-7f9ec38a703f/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8d274cdd-bf86-457d-a83c-7f9ec38a703f/Untitled.png)

String을 Event 타입으로 변환할 수 없기 때문에 500 에러와 함께

MethodArgumentConversionNotSupportedException이 발생한다.

그렇다면 정상적인 동작을 위해 PropertyEditor를 구현해보자.

PropertyEditor를 구현(inplement)해도 되지만, 구현해야하는 메소드가 상당히 많기 때문에

PropertyEditorSupport를 상속받아서 필요한 메소드만 구현한다.

현재 우리가 필요한 기능은 Text를 Event로 변환하는 작업이기 때문에 아래와 같이 구현할 수 있다.

(주석 참조)

```java
public class EventEditor extends PropertyEditorSupport {

    @Override
    public String getAsText() {
			// Object 타입인 getValue를 Event 타입으로 변환해준다.
        Event event = (Event)getValue();
			// id를 넘겨준다.
        return event.getId().toString();
    }

    @Override
    public void setAsText(String text) throws IllegalArgumentException {
			// 문자열로 들어오는 text를 Integer로 형변환해주고, 이를 Event 생성자에 전달한다.
			// setValue라는 메소드에 Event 객체를 넣어준다.
        setValue(new Event(Integer.parseInt(text)));
    }
}
```

> 여기에서 주의할 점은 getValue와 setValue는 상태정보를 저장하고 있기 때문에 Thread-safe 하지 않다. 그러므로 PropertyEditor의 구현체들은 여러 Thread에 공유해서 사용하면 안된다. 즉, Bean으로 등록해서 사용하면 안된다.
>

그렇다면 PropertyEditor를 어떻게 사용해야할까?

@InitBinder라는 애노테이션을 이용해 Controller에 등록해서 사용하는 방법이 있다.

DataBinder의 구현체 중 하나인 WebDataBinder를 통해 원하는 타입을 처리해 줄 PropertyEditor를

등록해서 사용할 수 있다.

```java
@RestController
public class EventController {
		// 추가된 부분
    @InitBinder
    public void init(WebDataBinder webDataBinder) {
        webDataBinder.registerCustomEditor(Event.class, new EventEditor());
    }

    @GetMapping("/event/{event}")
    public String getEvent(@PathVariable Event event) {
        System.out.println(event);
        return event.getId().toString();
    }
}
```

우리는 Event 타입을 처리하는 기능이 필요하기 때문에 다음과 같이 WebDataBinder를 통해 Event 타입을 처리할 PropertyEditor(EventEditor)를 등록해준 후 다시 Test를 돌려보자.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d9d23806-aac7-48bc-8ecc-3eeaedac4736/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d9d23806-aac7-48bc-8ecc-3eeaedac4736/Untitled.png)

테스트가 잘 통과 된것을 확인할 수 있다.

대략적인 흐름을 설명하면,

1. Controller가 어떤 요청을 처리하기 전에 Controller에 정의된(@InitBinder) DataBinder를 사용하게 된다.
2. DataBinder로 등록된 EventEditor의 setAsText 메소드를 호출한다.
3. 문자열로 들어온 {event}의 값을 숫자로 변환해서 Event 객체로 바꿔준다.
4. Test가 정상적으로 동작한다.

---

PropertyEditor는 고전적인 방법으로 구현 자체도 편리하지가 않고, Thread-safe하지 않기 때문에

Bean으로 등록해서 사용하기 위험하다. 따라서 스프링 3.0부터는 Converter와 Formatter같은 DataBinding과 관련된 인터페이스가 추가되었다.