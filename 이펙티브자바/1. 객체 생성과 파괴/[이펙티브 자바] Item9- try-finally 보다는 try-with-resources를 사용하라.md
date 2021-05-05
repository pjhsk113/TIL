# [이펙티브 자바] Item9- try-finally 보다는 try-with-resources를 사용하라

---

자바에는 close 메서드로 직접 닫아줘야하는 자원이 많다.

ex) InputStream, OutputStream, java.sql.Conncetion 등

자원 닫기는 클라이언트가 놓치기 쉬워 예측할 수 없는 성능 문제로 이어지기 때문에 자원을 닫는 수단으로 try-finally이 많이 사용됬다.

# 1. try-finally의 단점

### 1.1 코드 가독성이 떨어진다.

```java
String firstLineOfFile(String path) throw IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

가독성이 나쁘지않다. 하지만 자원이 하나 더 추가된다면?

```java
static void copy(String src, String dst) throws IOException{
  InputStream in = new FileInputStream(src);
  try{
    OutputStream out = new FileOutputStream(dst);
    try{
      byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while ((n = in.read(buf)) >= 0)
        out.write(buf, 0, n);
    }finally{
      out.close();
    }
  }finally{
    in.close();
  }
}
```

코드의 가독성이 떨어지고, 지저분해 진다.

### 1.2 두번째 예외가 첫번째 예외를 집어삼킨다.

readLine 메서드가 예외 > close 메서드 실패 > 두번째 예외가 첫번째 예외를 집어삼킴

위의 예제에서 firstLineOfFile 메서드안의 readLine 메서드에서 예외가 발생하면, 같은 이유로 close 메서드도 실패할 것이다. 이러한 상황이라면 두번째 예외가 첫번째 예외를 집어삼키는 일이 발생한다. 

위와 같은 상황에서 스택 추적 내역에는 두 번째 예외 내역만 남게되어 디버깅이 어려워진다.

# 2. try-finally의 대안

자바7에서 등장한 try-with-resources를 사용하면 위의 단점들을 모두 보완할 수 있다. 

이 구조를 사용하려면 AutoCloseable 인터페이스를 구현해야 한다. 단순히 close 메서드를 정의한 인터페이스이다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FctF2cu%2FbtqRAkfVpgo%2FwUoFwn6CWp5ljuhv6CajT0%2Fimg.png)

```java
String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(
	    		new FileReader(path))) {
        return br.readLine();
    }
}
```

```java
static void copy(String src, String dst) throws IOException {
  try (InputStream   in = new FileInputStream(src);
       OutputStream out = new FileOutputStream(dst)) {
    byte[] buf = new byte[BUFFER_SIZE];
    int n;
    while ((n = in.read(buf)) >= 0)
      out.write(buf, 0, n);
  } catch (IOException e) {
    return defaultVal;
  }
}
```

try-with-resoureces 구문을 적용한 예제이다.

try-finally 구문보다 가독성이 좋고, catch를 이용해 try문을 중첩하지 않고도 다수의 예외처리가 가능하다. 또한 숨겨진 예외들도 스택 추적 내역에 출력(suppressed)되어 디버깅에도 용이하다.

readLine()과 close() 호출 양쪽에서 예외가 발생한 경우에도 close() 예외는 숨겨지고 readLine()에서 발생한 예외가 기록된다.
