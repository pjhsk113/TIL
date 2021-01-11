# [이펙티브 자바] Item4- 인스턴스화를 막으려거든 private 생성자를 사용하라

---

# 정적 메서드, 정적 필드를 가지는 유틸리티 클래스

개발을 하다보면 **유틸리티 클래스**가 필요한 경우가 있다. 

이러한 도구용 클래스는 단순히 정적 메서드와 정적 필드만을 담은 클래스로써 계속해서 무거워질 수 있고, 객체지향과는 거리가 멀지만 분명 쓸모가 있다. 

예시로는 **java.lang.Math**와 **java.util.Arrays**가 있다.

Math 클래스는 계산과 관련된 **정적 메서드**와 **상수**들을 담고 있고,

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/615a611b-f3df-4d12-b78e-5b062b9bff96/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210111%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210111T101933Z&X-Amz-Expires=86400&X-Amz-Signature=130f8958ae9847f501c3e3c028d3ed0b9c76defe8490e177dc421d05f4363465&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

 Arrays 클래스는 배열과 관련된 **정적 메서드**를 담고 있다. 

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/d003278d-6fc1-4637-b4f2-c79deee8b970/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210111%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210111T101935Z&X-Amz-Expires=86400&X-Amz-Signature=faa6bbd0ef9cbd0c7d1d063da9a46085e8b6f20081ae2f363afd2747a735d0b7&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

이러한 유틸리티 클래스는 **인스턴스화를 위해 설계된 클래스가 아니다.** 따라서 두 클래스 모두 private 생성자를 가지는 것을 볼 수 있다.

# private 기본 생성자를 명시하는 이유

그렇다면 왜 기본 생성자를 private으로 명시해주는 것일까?

그 이유는 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들기 때문이다. 즉, 매개변수를 받지  않는 public 생성자가 만들어지고, 의도치않게 인스턴스화를 허용할 수 있게 된다.

아래의 예시는 String으로 입력된 값을 split하고 List<Integer>형태로 변환해주는 유틸 클래스이다. 

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/ed07d59c-d1d7-4e44-8480-46b6bd2fd247/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210111%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210111T101937Z&X-Amz-Expires=86400&X-Amz-Signature=6985d86c65bda98f06630681ae0eac69b15ffbb00cf740394aa2390d4636d5b2&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

이 클래스를 컴파일해보면 다음과 같이 public 기본 생성자가 자동으로 생성된 것을 볼 수 있다.

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/dcc9f6f6-4bcd-45e3-8f1b-1cd4e967f861/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210111%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210111T101939Z&X-Amz-Expires=86400&X-Amz-Signature=ede33c321d296c2928fd9bf0af25ccf29976b6aa2ed8ad0a437860d8d958d3d8&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

인스턴스화를 위해 설계된 클래스가 아님에도 인스턴스화를 할 수 있게 된 것이다. 

이를 방어하기 위해 기본 생성자를 private로 명시해주면

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/3b6a6ca4-ec60-4e4d-b583-99ba581b4926/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210111%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210111T101941Z&X-Amz-Expires=86400&X-Amz-Signature=42094a056f9b9395f1f9c782f01a813519eafac6b40e3fe211e1077770f21975&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/3208c6ae-e463-4fa1-80dc-91672ec86e65/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210111%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210111T101942Z&X-Amz-Expires=86400&X-Amz-Signature=b19a8a8ccf612dd04c82f1e58acdcfc5206616bbe81789de7a575bc1437fa62f&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

기본 생성자가 private로 생성된 것을 볼 수 있다.

# 핵심 정리

추상클래스는 인스턴스화가 불가능하기 때문에 인스턴스화를 막기위한 방법으로 추상클래스를 떠올릴 수 있다.

하지만 인스턴스화를 막기위해 추상클래스를 만든다면, 오히려 더 큰 문제가 될 수 있다. 

사용자입장에서 추상클래스를 보고 **상속하여 사용하라**는 의미로 받아들일 수 있기 때문이다.

그러니 인스턴스화를 막으려면 위에 설명한 방법처럼 private 생성자를 추가하자. 그리고 주석을 통해 생성자 역할에 대한 직관성을 높이고, 더 신경쓴다면 AssertionError를 에러를 던져주자.

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/1f1b235b-2c06-4dd1-85c9-d3b45bb3c988/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210111%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210111T101945Z&X-Amz-Expires=86400&X-Amz-Signature=0cce1a37ee1ad5444953200d3285fd1f7e7d17bc78904d19dcf055916feb04e4&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)
