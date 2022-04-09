# [스프링 프레임워크 핵심기술] - Null Safety

---

> 백기선님의 **스프링 프레임워크 핵심 기술**이라는 강좌를 들으며 공부한 내용을 정리한 글입니다.
>

스프링 5에서 다음과 같은 Null 관련 애노테이션이 추가되었다.

- @NonNull
    - Null 허용하지 않음
- @Nullable
    - Null 허용
- @NonNullApi
    - 패키지 레벨에 모든 파라미터와 리턴에 @NonNull을 적용
- @NonNullFields
    - 패키지 레벨에서 필드에 Null 허용하지 않음

위의 애노테이션들을 사용하면 IDE의 도움을 받아 컴파일 타임에서 최대한 NullPointerException을 방지할 수 있다.

# 1. Null-Safety

먼저 간단한 예를 위해 Service와 AppRunner를 만들어보자.

```java
@Service
public class EventService {
		// return 타입이 NonNull
    @NonNull
    public String createEvent(@NonNull String name) {
        return "hello " + name;
    }
}
```

```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    EventService eventService;

    @Override
    public void run(ApplicationArguments args) throws Exception {
				// 파라미터 값을 null로 보내본다.
        eventService.createEvent(null);
    }
}
```

보통 잘못된 메서드명을 입력하면 아래 사진처럼 IDE에서 경고를 해준다.

![ex) 메서드명에 오타가 난 경우](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fc752bc7-ace4-4b1c-9882-7196814c7b45/Untitled.png)

ex) 메서드명에 오타가 난 경우

@NonNull 애노테이션은 파라미터 값으로 Null이 입력되었을 경우, 리턴 값이 Null일 경우에

IDE의 경고 메시지의 도움을 받아 Null을 방지한다.

아래의 사진처럼 @NonNull 애노테이션을 명시해서 Null값을 받지 않겠다고 선언해주었다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eff936b3-6f70-4b96-ba75-d8b932faa034/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eff936b3-6f70-4b96-ba75-d8b932faa034/Untitled.png)

의도대로라면 IDE의 경고 메시지를 통해 Null 방지를 위한 기능이 작동해야한다.

하지만 아래의 사진처럼 현재 값을 Null로 보내고있음에도 아무런 경고 메시지가 뜨지않는다.

Null 방지를 위한 기능이 작동하지 않는 것이다.

![ex) 파라미터 값을 null로 보낸 경우](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6bcd66f3-970d-4877-972f-22d96517afa4/Untitled.png)

ex) 파라미터 값을 null로 보낸 경우

이를 제대로 작동하게 하기 위해서는 Compiler 설정을 해주어야한다. (IntelliJ Ultimate기준)

1. 먼저 Ctrl + Alt + S를 눌러 IDE Settings을 호출한다.

2. Build, Execution, Deployment → Compiler → CONFIGURE ANNOTATIONS 선택

   ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f83b9257-d3c1-49c6-af29-8041d7c1d1b9/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f83b9257-d3c1-49c6-af29-8041d7c1d1b9/Untitled.png)

3. Springframework의 애노테이션을 추가해준다.

   ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a12a7796-04f8-4e27-bdcd-da3d9b82c940/Untitled1.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a12a7796-04f8-4e27-bdcd-da3d9b82c940/Untitled1.png)

   ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8163a135-1791-412b-a9a3-97c56c484341/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8163a135-1791-412b-a9a3-97c56c484341/Untitled.png)

   ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/11171a7d-c252-4923-adb6-5c41e04fed29/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/11171a7d-c252-4923-adb6-5c41e04fed29/Untitled.png)

   ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/983cd4d2-98bb-49f6-aa63-c2a1691a06f0/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/983cd4d2-98bb-49f6-aa63-c2a1691a06f0/Untitled.png)


다시 화면으로 돌아와 아까 작성했던 AppRunner를 살펴보면,

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a98397b2-91c9-458c-9dc5-c268f20e0fcb/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a98397b2-91c9-458c-9dc5-c268f20e0fcb/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ffe62834-bc51-433a-99b1-a35f784de07a/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ffe62834-bc51-433a-99b1-a35f784de07a/Untitled.png)

다음과 같이 Null 방지 에러메시지가 출력되는 것을 확인할 수 있다.