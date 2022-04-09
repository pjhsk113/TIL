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

![ex) 메서드명에 오타가 난 경우](https://blog.kakaocdn.net/dn/7Acuh/btqERQX5BqJ/mkgZuyCLeM8ZXEZMqHKick/img.png)

ex) 메서드명에 오타가 난 경우

@NonNull 애노테이션은 파라미터 값으로 Null이 입력되었을 경우, 리턴 값이 Null일 경우에

IDE의 경고 메시지의 도움을 받아 Null을 방지한다.

아래의 사진처럼 @NonNull 애노테이션을 명시해서 Null값을 받지 않겠다고 선언해주었다.

![](https://blog.kakaocdn.net/dn/osUY9/btqEQYP53qn/5oKMLL7OSC9rqZizGMcox0/img.png)

의도대로라면 IDE의 경고 메시지를 통해 Null 방지를 위한 기능이 작동해야한다.

하지만 아래의 사진처럼 현재 값을 Null로 보내고있음에도 아무런 경고 메시지가 뜨지않는다.

Null 방지를 위한 기능이 작동하지 않는 것이다.

![ex) 파라미터 값을 null로 보낸 경우](https://blog.kakaocdn.net/dn/bpQRwv/btqEPmkgI4L/vuoXybooT0JKTYKX7iaO9k/img.png)

ex) 파라미터 값을 null로 보낸 경우

이를 제대로 작동하게 하기 위해서는 Compiler 설정을 해주어야한다. (IntelliJ Ultimate기준)

1. 먼저 Ctrl + Alt + S를 눌러 IDE Settings을 호출한다.

2. Build, Execution, Deployment → Compiler → CONFIGURE ANNOTATIONS 선택

   ![](https://blog.kakaocdn.net/dn/FJy0a/btqEPmxOk6K/8fX72m8QOL9R5Bhpiojv3K/img.png)

3. Springframework의 애노테이션을 추가해준다.

   ![](https://blog.kakaocdn.net/dn/br3asV/btqEPvafrfT/fKuEyx3XhEikN2z6yGu5KK/img.png)

   ![](https://blog.kakaocdn.net/dn/F5OAo/btqEPuCn4TC/gsed2RntJ9Jkx2RWDUbSl0/img.png)

   ![](https://blog.kakaocdn.net/dn/d3hJFl/btqEPU1TrRs/F6KUAms1okFSNN6E33jppK/img.png)

   ![](https://blog.kakaocdn.net/dn/oATQl/btqERRvWcs2/0WdYq3bnB19JWeDUbV0KJK/img.png)


다시 화면으로 돌아와 아까 작성했던 AppRunner를 살펴보면,

![](https://blog.kakaocdn.net/dn/3OUug/btqEQYieLEU/gJ1OaCJxKojhduZhO8ihXK/img.png)

![](https://blog.kakaocdn.net/dn/XYQGT/btqERsQFKnf/ZdDvAlAn6WWkTlPdl3vw6K/img.png)

다음과 같이 Null 방지 에러메시지가 출력되는 것을 확인할 수 있다.