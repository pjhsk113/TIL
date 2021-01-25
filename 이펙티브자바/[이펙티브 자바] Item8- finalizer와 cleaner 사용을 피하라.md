# [이펙티브 자바] Item8- finalizer와 cleaner 사용을 피하라

---

자바의 객체와 관련된 자원 회수는 가비지 컬렉터가 담당하고있다. 따라서 프로그래머에게 객체 자원 수거에 대한 아무런 작업도 요구하지 않지만, 그럼에도 자바는 두 가지 객체 소멸자를 가지고있다.

> 비메모리 자원 회수는 try-with-resources와 try-finally를 사용해 해결한다.

### finalizer

- 예측할 수 없고, 상황에 따라 위험할 수 있어 보통 불필요하다.

### cleaner

- finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고 느리다. 이 역시 보통은 불필요하다.

# 두 가지의 객체 소멸자는 예측할 수 없다.

finalizer와 cleaner는 호출 후 즉시 수행된다는 보장이 없다. 즉, 호출된 후 언제 실행될지 알 수 없다는 의미이다. finalizer와 cleaner가 실행되기까지 얼마나 걸릴지 알 수 없기 때문에, 제때 실행되어야 하는 작업은 절대 할 수 없다. 

### ex) 파일 닫기

시스템이 동시에 열 수 있는 파일 개수에는 한계가 있는데, finalizer와 cleaner가 언제 실행될지 몰라 파일을 계속 열어 둔다면? 새로운 파일을 열지 못하게 되어 프로그램이 실패할 수 있다.

### ex) 낮은 우선 순위

**finalizer 스레드**가 **다른 애플리케이션 스레드보다 우선순위가 낮을 시 문제**가 발생할 수 있다. 객체 수천 개가 finalizer 대기열에서 회수되기만을 기다리다 프로그램이 죽어버릴 수 있다.

cleaner는 자신을 수행할 스레드를 제어할 수 있다는 면에서 조금 낫지만, 여전히 백그라운드에서 **가비지 컬렉터에게 제어되기** 때문에 **즉각 수행되리라는 보장은 없다.**

# 수행 시점, 수행 여부를 보장하지 않는다.

**접근할 수 없는 객체에 대한 종료 작업을 수행하지 못하고 프로그램이 중단될 수 있다.** 따라서 프로그램 라이프 사이클과 상관없는, 상태를 영구적으로 수정하는 작업에 대해서는 finalizer나 cleaner에 의존해서는 안된다.

DB와 같은 공유 자원의 영구 Lock 해제를 finalizer나 cleaner에 의존하게 되면 분산 시스템 전체가 멈출 수 있다.

# 예외 시 문제점

 finalizer 동작 중 예외가 발생하면 그 예외는 무시되며 처리할 작업이 남았더라도 종료된다. 따라서 해당 객체는 훼손될 수 있고, 다른 스레드가 훼손된 객체에 접근하여 사용하려 한다면 이에 따른 동작 예측을 할 수 없다.

보통의 경우엔 스레드를 중단시키고 스택 추적 내역을 출력하지만, 위의 경우에는 경고도 출력하지 않는다.

cleaner는 자신의 스레드를 통제할 수 있기 때문에 위와 같은 문제는 발생하지 않는다.

# 성능 문제

 finalizer나 cleaner는 가비지 컬렉터의 효율을 떨어트리기 때문에 성능에 문제가 존재한다.

# finalizer attack 보안 문제

finalizer attack 예시

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcKFYLC%2FbtqRCWeLeNh%2FNzsEEcsvF9DaZrXFRT544K%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fsv2Rv%2FbtqRCWlvlNJ%2FRMm5HP7OF5U9JMgfwinQe1%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbTpoqo%2FbtqRI7050Ic%2FomYxKBKxh1Kb2tkSKtxfM1%2Fimg.png)

생성자나 직렬화 과정에서 예외가 발생하면 finalizer가 수행되는데, 이 finalizer를 악의적으로 오버라이딩한 하위클래스의 finalizer가 수행될 수 있다. 심지어 이 finalizer를 정적필드에 할당하면 가비지컬렉터에의해 수거되지도 않는다.

## 해결 방법

**initialized flag**
객체가 정상적으로 생성될 때 flag값을 셋팅하고 메서드 호출 때마다 제일 먼저 저 flag 값을 검사하는 방법이다. 이 코딩 기법은 작성하기 번거롭고 실수로 생략하기 쉽다. 그리고 subclassing을 통한 공격을 막을 수는 없다.

**subclssing 금지**
클래스를 final로 선언함으로써 subclass 자체를 막아버린다. 하지만 이 기법은 확장성 자체를 없애버린다.

**final finalizer**
final finalizer 메서드를 생성함으로써 subclass에서 finalizer 메서드 재사용과 오버라이딩을 금지시킨다.

# finalizer와 cleaner의 대안

### AutoCloseable을 구현한다.

파일이나 스레드 등 종료해야 할 자원에 AutoCloseable을 구현하고, 인스턴스를 다 쓰고 난 후 close 메서드를 호출하면 된다.

이때, 예외가 발생하면 제대로 종료되도록 try-with-resources를 사용해야 한다.

close 메서드 호출여부를 필드로 저장하고, 객체 사용 시 필드를 검사해서 이미 닫혔으면 IllegalStateException을 던지도록 구현하면 좋다.

## 예제

FileInputStream의 close 메서드를 살펴보면,

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FqTaqv%2FbtqRt6Qge9D%2FR6ZbxJybcdm5WSsUMe3A3k%2Fimg.png)

closed 변수를 통해 객체의 유효 여부를 기록한다. 다른 메서드는 이 필드를 검사해서 객체가 닫힌 후에 불렸다면 Exception를 던진다.

# 그렇다면 finalizer와 cleaner는 언제 사용할까?

### 1. AutoCloseable을 구현하지 않았을 경우를 대비한 안전망 역할일때 사용

cleaner나 finalizer가 즉시 호출되어 자원을 회수한다는 보장은 없지만, 클라이언트가 하지 않은 자원 회수를 늦게라도 해주는 것이 안하는 것보다 났다. 이러한 안전망 역할이 필요할 경우 사용한다.

자바 라이브러리 중 FileInputStream, FileOutputStream, ThreadPoolExecutor가 안전망 역할의 finalizer를 제공하는 대표적인 예이다.

### 2. 네이티브 피어와 연결된 객체

> 네이티브 피어란?
일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다.

네이티브 피어는 자바 객체가 아니기 때문에 GC의 대상이 되지 않는다. 따라서 자바 피어를 회수할 때 네이티브 객체까지 회수하지 못한다. 이때 finalizer나 cleaner가 필요하다. 

다만, 성능 저하가 있을 수 있으므로 성능 저하를 감당하지 못하거나, 즉시 자원 회수가 필요하다면  close 메서드를 사용해야한다.
