# [이펙티브 자바] Item4- 인스턴스화를 막으려거든 private 생성자를 사용하라

---

# 정적 메서드, 정적 필드를 가지는 유틸리티 클래스

개발을 하다보면 **유틸리티 클래스**가 필요한 경우가 있다. 

이러한 도구용 클래스는 단순히 정적 메서드와 정적 필드만을 담은 클래스로써 계속해서 무거워질 수 있고, 객체지향과는 거리가 멀지만 분명 쓸모가 있다. 

예시로는 **java.lang.Math**와 **java.util.Arrays**가 있다.

Math 클래스는 계산과 관련된 **정적 메서드**와 **상수**들을 담고 있고,

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FchQZMS%2FbtqNqdMs6ni%2FkqnpgXKbamXGYxUep6Bqr0%2Fimg.png)

 Arrays 클래스는 배열과 관련된 **정적 메서드**를 담고 있다. 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FSbHb6%2FbtqNqCLYVfe%2FUgykzrFSUievbGdjZ4kylK%2Fimg.png)

이러한 유틸리티 클래스는 **인스턴스화를 위해 설계된 클래스가 아니다.** 따라서 두 클래스 모두 private 생성자를 가지는 것을 볼 수 있다.

# private 기본 생성자를 명시하는 이유

그렇다면 왜 기본 생성자를 private으로 명시해주는 것일까?

그 이유는 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들기 때문이다. 즉, 매개변수를 받지  않는 public 생성자가 만들어지고, 의도치않게 인스턴스화를 허용할 수 있게 된다.

아래의 예시는 String으로 입력된 값을 split하고 List<Integer>형태로 변환해주는 유틸 클래스이다. 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FrJvQz%2FbtqNpG2nENf%2FV05UcyPOXKglGSHMUDUYY0%2Fimg.png)

이 클래스를 컴파일해보면 다음과 같이 public 기본 생성자가 자동으로 생성된 것을 볼 수 있다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbi2cPN%2FbtqNtilJdXh%2F3l6V6OIjTAlHRz0DxHOgV1%2Fimg.png)

인스턴스화를 위해 설계된 클래스가 아님에도 인스턴스화를 할 수 있게 된 것이다. 

이를 방어하기 위해 기본 생성자를 private로 명시해주면

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbc4M4y%2FbtqNqekeKOq%2FcqBJokMlIihxSNUKBPE5MK%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbnFJkO%2FbtqNx8WUU6K%2FAKdXqm4yaZZ9D0ZhGrkxo1%2Fimg.png)

기본 생성자가 private로 생성된 것을 볼 수 있다.

# 핵심 정리

추상클래스는 인스턴스화가 불가능하기 때문에 인스턴스화를 막기위한 방법으로 추상클래스를 떠올릴 수 있다.

하지만 인스턴스화를 막기위해 추상클래스를 만든다면, 오히려 더 큰 문제가 될 수 있다. 

사용자입장에서 추상클래스를 보고 **상속하여 사용하라**는 의미로 받아들일 수 있기 때문이다.

그러니 인스턴스화를 막으려면 위에 설명한 방법처럼 private 생성자를 추가하자. 그리고 주석을 통해 생성자 역할에 대한 직관성을 높이고, 더 신경쓴다면 AssertionError를 에러를 던져주자.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbxwC1Y%2FbtqNx92A2Ls%2FjfEmivwi6kbH4Qvl4FGkEk%2Fimg.png)
