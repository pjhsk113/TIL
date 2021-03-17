# [이펙티브 자바] Item25- 톱레벨 클래스는 한 파일에 하나만 담으라

## 문제점

소스 파일 하나에 여러개의 톱레벨 클래스를 선언하더라도 컴파일에는 문제가 없다. 하지만 이는 심각한 위험을 감수해야하는 행위이다. 

소스 파일 하나에 여러개의 톱레벨 클래스를 선언함으로 컴파일 에러는 발생하지 않지만 예상치 못한 결과가 나타날 수 있다. 컴파일러에 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라지기 때문이다.

## 예시

아래와 같이 **Utensil.java** 파일에 Utensil과 Dessert 클래스가 함께 존재한다고 가정해보자. 

```java
// Utensil.java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
```

그러던 중 우연히 위와 똑같은 클래스를 담은 **Dessert.java** 파일을 생성했다고 해보자.

```java
// Dessert.java
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
```

아래의 Main을 실행했을 때 어떤일이 벌어질까?

```java
public class Main {
	public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);
	}
}
```

일반적인 경우라면 중복 클래스가 존재한다는 컴파일 에러가 발생할 것이다. 

하지만 `javac Main.java Utensil.java` 의 명령으로 컴파일을 수행하면 "pancake"가 출력될 것이고 `javac Dessert.java Main.java` 명령으로 컴파일을 수행하면 "potpie"가 출력될 것이다. 

## 해결 방안

해결책은 간단하다. **정적 멤버 클래스**로 만들거나 **톱레벨 클래스를 서로 다른 소스 파일로 분리**하면 된다.

톱레벨 클래스를 한파일에 담고 싶다면 **정적 멤버 클래스**를 사용하면 된다.

### AS-IS

```java
public class Main {
	public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);
	}
}
```

```java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
```

### TO-BE

```java
// 정적 멤버 클래스를 사용한 예제
class Main {
  public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);
	}

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }

}
```