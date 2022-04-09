# [스프링 프레임워크 핵심기술] - MessageSource

---

> 백기선님의 **스프링 프레임워크 핵심 기술**이라는 강좌를 들으며 공부한 내용을 정리한 글입니다.
>

이번에 다룰 내용은 ApplicationContext의 다양한 기능 중 한 가지인 MessageSource이다.

ApplicationContext는 다양한 기능을 상속하고있는데, 그 중 MessageSource는 다국어 처리를 할 때 사용되는 객체이다.  간단한 예제로 MessageSource의 사용 방법을 알아보자.

---

# 1. 사용 예제

테스트를 위해 먼저 다국어 처리를 해줄 메세지를 만들어야 한다.

스프링 부트에서는 별다른 설정 없이 messages로 시작하는 properties들을 MessageSource로 읽어알아서 Bundle로 인식한다.

![](https://blog.kakaocdn.net/dn/b3FL5h/btqEChXCnYu/5DLTqlIO4ionHivUym0P2k/img.png)

다음과 같이 resources 밑에 **message.properties**와 **message_ko_KR.properties**를 만들어준다.

```yaml
# message.properties
greeting=Hello, {0}
```

```yaml
# message_ko_KR.properties
greeting=안녕, {0}
```

<aside>
💡 원래는 Bean으로 등록해주어야 정상적으로 사용할 수 있지만, 스프링부트에서는 ResourceBundleMessageSource라는 Bean이 이미 등록되어있어서 messages라는 이름을 가진 리소스를 자동으로 읽어 Bundle로 만들어준다. 따라서 별다른 설정없이 사용할 수 있는 것이다.

</aside>

messages 리소스를 만들었다면, Runner에서 간단히 테스트를 해볼 수 있다.

```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    MessageSource messageSource;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Locale.setDefault(new Locale("en","US"));
        System.out.println(messageSource.getMessage("greeting", new String[]{"기록하는 습관"},Locale.getDefault()));
        System.out.println(messageSource.getMessage("greeting", new String[]{"기록하는 습관"},Locale.KOREA));
    }
}
```

MessageSource를 주입받아 getMessage를 이용해 콘솔에 메시지를 찍어보자.

![](https://blog.kakaocdn.net/dn/3tGdT/btqECu3uXB3/ToELxoB1DTa2vijTUauiOK/img.png)

Locale에 따라 잘 찍힌다.

이외에도 MessageSource의 인스턴스를 ReloadableResourceBundleMessageSource로 생성하여

Bean으로 등록하면, 어플리케이션 동작중에 메시지가 변환되어도 Build만 해주면

메시지가 reload되는 기능을 사용할 수 있다.

---

사실 EnvironmentCapable, MessageSource 같은 IoC 컨테이너에 대한 세부 내용은 정리하지

않으려 했었다. 잘 와닿지 않았을뿐더러, 간단한 사용 방법과 개념만 소개하는 강의였기 때문이다.

각각의 역할에 대한 이해만 하고 넘어가야지 하고있었는데 MessageSource 강의를 듣고 이건

정리해야겠다는 생각이 들었다.

현재 회사에서 다국어 처리를 사용하고 있는데, 사실 별 다른 이해없이 사용하고 있었다.

"이거 그냥 이렇게 쓰면 되네?" 결국 사용법만 대충 익혀서 사용하고 있었던것이다.

선배분들이 잘 설계해놓은 개발 코드들을 그냥 가져다쓴 것 뿐이면서, 마치 내가 이 기술을

이해하고 내 것으로 만들었다는 착각에 빠져있었다. 즉,  이 로직이 어떻게 흘러가고 어떻게

처리되는지에 대한 이해가 하나도 없었다. 그냥 사용 방법만 알뿐.

이번 강의를 들으면서 내가 맨날 사용하고있던게 MessageSource였다는 걸 알았을 때는

조금 어이가 없었다. 내가 지금까지 사용하던게 MessageSource를 이용한 기술이라니.

"내가 뭘 모르는지조차 모르는구나." 라는 생각이 들었다

이번 강의를 정리하는 글은 짧지만, 얻은건 이전 정리글들보다 더 많은 것 같다.

이번을 계기로 성장 욕구를 더 불태워봐야겠다.