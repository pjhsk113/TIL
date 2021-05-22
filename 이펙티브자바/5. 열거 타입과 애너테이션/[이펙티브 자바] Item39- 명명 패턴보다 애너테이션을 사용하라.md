# [이펙티브 자바] Item39- 명명 패턴보다 애너테이션을 사용하라

명명 패턴이란 변수나 함수의 이름을 일관된 방식으로 작성하는 패턴을 말한다. 이러한 **명명 패턴**은 전통적으로 도구나 프레임워크에서 특별히 다뤄야 할 프로그램 요소에 구분을 위해 사용되어 왔다.  

예를들면 JUnit3 에서는 테스트 메서드 이름을 test로 시작하게끔 했다. 이러한 명명 패턴 방식은 효과적이지만 단점도 크다.

# 명명 패턴의 단점

## 1. 오타가 나면 안 된다.

만약 test로 시작되어야 할 메서드 이름이 오타로 인해 tset로 작성되었다면, 명명 패턴에는 벗어나지만 프로그램 상에서는 문제가 없기 때문에 테스트 메서드로 인식하지 못하고 테스트를 수행하지 않는다.

## 2. 명명 패턴을 의도한 곳에서만 사용할 거라는 보장이 없다.

개발자는 JUnit3의 명명 패턴인 **'test'**를 메서드가 아닌 **클래스의 이름**으로 지음으로써 해당 클래스의 모든 테스트 메서드가 수행되길 바랄 수 있다. 하지만 JUnit은 클래스 이름에는 관심이 없다. 따라서 개발자가 의도한 테스트는 전혀 수행되지 않는다.

## 3. 명명 패턴을 적용한 요소를 매개변수로 전달할 마땅한 방법이 없다.

특정 예외를 던져야 성공하는 테스트가 있을 때, 메서드 이름에 포함된 문자열로 예외를 알려주는 방법이 있지만 보기 흉할 뿐 아니라 컴파일러가 문자열이 예외 이름인지 알 도리가 없다.

애너테이션을 사용하면 위의 단점을 모두 해결할 수 있다. 그러므로 많은 단점을 가진 명명 패턴을 사용하기 보다는 **애너테이션을 사용하자.**

# 마커 애너테이션

마커 애너테이션은 **아무 매개변수 없이 단순히 대상에 마킹하는 용도**로 사용하는 애너테이션이다.

```java
/**
* 테스트 메서드임을 선언하는 애너테이션이다.
* 매개변수 없는 정적 메서드 전용
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test{}
```

### 메타 애너테이션

애너테이션 선언에 다는 애너테이션을 말한다. 위 예제에서는 @Retention과 @Target이 메타 애너테이션에 해당한다.

- @Retention(RetentionPolicy.RUNTIME)  - 보존 정책
    - @Test가 런타임에도 유지되어야 한다는 표시
- @Target(ElementType.METHOD) - 적용 대상
    - @Test가 반드시 메서드에 선언되어야 한다는 표시

이러한 마커 애너테이션은 적절한 애너테이션 처리기가 필요하다. 마커 애너테이션은 단순히 Test라는 이름에 오타가 있거나 메서드 선언 외의 프로그램 요소에 달면 컴파일 오류를 내어주는 역할을 할 뿐이다. 실제 클래스가 동작하는데 직접적인 영향을 미치는게 아니고 유용한 정보를 제공할 뿐이다.

```java
// 마커 애너테이션 처리기
public class RunTests {
  public static void main(String[] args) throws Exception {
    int tests = 0;
    int passed = 0;
    Class<?> testClass = Class.forName(args[0]);
    for (Method m : testClass.getDeclaredMethods()) {
      if (m.isAnnotationPresent(Test.class)) {
        tests++;
        try {
          m.invoke(null);
          passed++;
        } catch (InvocationTargetException wrappedExc) {
          Throwable exc = wrappedExc.getCause();
          System.out.println(m + " 실패: " + exc);
        } catch (Exception exc) {
          System.out.println("잘못 사용한 @Test: " + m);
        }
      }
    }
    System.out.printf("성공: %d, 실패: %d%n",
                      passed, tests - passed);
  }
}
```

리플렉션을 이용하여 마커 애너테이션을 찾고, 예외 발생시 InvocationTargetException으로 Wrapping된다. 그래서 해당 예외에 담긴 실패 정보를 추출해서 출력한다. 

해당 애너테이션의 의도와 다르게 사용되어졌을 경우**(매개변수 없는 정적 메서드가 아닌 경우)**에는 InvocationTargetException 외의 다른 예외가 발생되는데, 이를 처리하기 위해서는 새로운 애너테이션 타입이 필요하다.

# 매개변수를 가진 애너테이션

```java
// 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
  Class<? extends Throwable> value();
}
```

이 애너테이션의 매개변수 타입은 Class<? extends Throwable>이다. Throwable을 확장한 클래스의 Class 객체를 뜻한다. 즉, 모든 예외 타입을 수용한다는 뜻이다.

사용 방법은 다음과 같다.

```java
public class Sample {
	// 성공해야한다.
	@ExceptionTest(ArithmeticException.class)
	public static void m1() {
		int i = 0;
		i = i / i;
	}

	// 실패해야한다. (다른 예외 발생)
	@ExceptionTest(ArithmeticException.class)
	public static void m2() {
		int[] a = new int[0];
		int i = a[1];
	}
}
```

위와 같이 발생할 예외를 인자로 넘겨주면 된다. 애너테이션 처리기에서는 다음과 같이 코드를 수정한다.

```java
// 애너테이션 처리기
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (InvocationTargetException wrappedEx) {
                    Throwable exc = wrappedEx.getCause();
                    Class<? extends Throwable> excType =
                            m.getAnnotation(ExceptionTest.class).value();
                    if (excType.isInstance(exc)) {
                        passed++;
                    } else {
                        System.out.printf(
                                "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
                                m, excType.getName(), exc);
                    }
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @ExceptionTest: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n",
                passed, tests - passed);
    }
}
```

더 나아가 예외를 여러개 명시하고 그 중 하나가 발생하면 성공하게 만들 수도 있다. 기존 애너테이션에 Class 객체를 배열로 수정해보자.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Exception>[] value();
}
```

그리고 애너테이션 처리기의 구현을 다음과 같이 변경한다.

```java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (Throwable wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    Class<? extends Throwable>[] excTypes =
                            m.getAnnotation(ExceptionTest.class).value();
                    for (Class<? extends Throwable> excType : excTypes) {
                        if (excType.isInstance(exc)) {
                            passed++;
                            break;
                        }
                    }
                    if (passed == oldPassed)
                        System.out.printf("테스트 %s 실패: %s %n", m, exc);
                }
            }
        }
    }
}
```

반복문을 추가하여 Throwable 배열을 검증하는 로직이 추가되었다.

```java
public class Sample {
	// 성공해야한다.
	@ExceptionTest(ArithmeticException.class)
	public static void m1() {
		int i = 0;
		i = i / i;
	}

	// 성공해야한다. (다른 예외 발생)
	@ExceptionTest({ArithmeticException.class, ArrayIndexOutOfBoundsException.class})
	public static void m2() {
		int[] a = new int[0];
		int i = a[1];
	}
}
```

이제 m2의 테스트도 통과할 수 있게 되었다.

# 반복 가능한 애너테이션

Java8 부터는 단일 요소에 애너테이션을 반복적으로 달 수 있는 @Repeatable 메타 애너테이션을 제공한다. 따라서 배열 매개변수 대신 @Repeatable 메타 애너테이션을 달면 단일 요소에 반복적으로 적용할 수 있게 되었다.

## @Repeatable 사용시 주의점

### 1. @Repeatable을 명시한 애너테이션을 반환하는 **컨테이너 애너테이션**을 하나 더 정의해야한다. 그리고 @Repeatable에 해당 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야한다.

### 2. 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야한다.

### 3. 적절한 @Retention과 @Target을 명시해야한다. 그렇지 않으면 컴파일 되지 않는다.

아래의 코드는 @Repeatable 메타 애너테이션을 적용한 ExceptionTest 애너테이션 선언부를 나타낸다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
  Class<? extends Throwable> value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
  ExceptionTest[] value();
}
```

적용 방법

```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() { }
```

@ExceptionTest를 여러 개 달면 하나만 달렸을 때와 구분하기 위해 컨테이너 애너테이션 타입이 적용된다. 애너테이션 처리기에서 이 둘을 구분하기 위해서는 isAnnotationPresent로 검사를 수행해야한다. isAnnotationPresent는 이 둘을 명확히 구분할 수 있기 때문이다.

```java
// 처리기
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(ExceptionTest.class)
                    || m.isAnnotationPresent(ExceptionTestContainer.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (Throwable wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    ExceptionTest[] excTests =
                            m.getAnnotationsByType(ExceptionTest.class);
                    for (ExceptionTest excTest : excTests) {
                        if (excTest.value().isInstance(exc)) {
                            passed++;
                            break;
                        }
                    }
                    if (passed == oldPassed)
                        System.out.printf("테스트 %s 실패: %s %n", m, exc);
                }
            }
        }
    }
}
```

이러한 방법은 코드 가독성을 높일 수 있지만 애너테이션을 처리하는 부분의 코드가 늘어나고 복잡해서 오류가 발생할 확률이 커진다는 것을 명심하자. 하지만 이번 예제는 이를 감수하더라도 명명 패턴보다 애너테이션이 좋다는 것을 확실히 보여준다.

# 핵심 정리

**애너테이션으로 처리할 수 있다면 명명 패턴을 사용할 이유는 없다.** 

어떤 도구 제작자를 제외하고는 애너테이션 타입을 직접 정의할 일은 거의 없다. 하지만 **자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용하자.**