# [이펙티브 자바] Item63 - 문자열 연결은 느리니 주의하라

문자열 연결 연산자(+)는 여러 문자열을 하나로 합쳐주는 편리한 수단이지만, 성능 저하가 상당하다. 문자열 연결 연산자로 문자열 n개를 잇는 작업은 n^2에 비례한다.

# 예시

청구서의 item을 전부 하나의 문자열로 연결하는 메서드가 있다.

```java
// 문자열 연결을 잘못 사용한 예 - 느리다!
public String statement() {
    String result = "";
    for (int i = 0; i < numItems(); i++) {
        result += lineForItem(i);
    }
    return result;
}
```

각 item의 원소 개수만큼 문자열을 잇는다. 품목의 개수가 많아지면 많아질 수록 성능 저하가 심해진다.

성능을 포기하고 싶지 않다면 String 대신 StringBuilder를 사용하자!

```java
// 문자열 연결 성능이 크게 개선된다!
public String statement2() {
    StringBuilder sb = new StringBuilder(numItems() * LINE_WIDTH);
    for (int i = 0; i < numItems(); i++) {
        sb.append(lineForItem(i));
    }
    return sb.toString();
}
```

statement() 메서드 수행 시간은 Item 수의 제곱에 비례해 늘어나고, statement2() 메서드의 수행시간은 선형으로 늘어나므로 Item이 많이질 수록 그 성능 차이도 심해질 것이다. 

## 테스트 결과

Item의 원소를 10000개로 하고 문자열의 길이가 80인 문자열의 연결 테스트해보니 엄청난 성능차이가 나타났다.

![](https://blog.kakaocdn.net/dn/bZFcBW/btrb9G36y7U/CKCHTyhjcBpEL6n7Sw7vT0/img.png)

# 핵심 정리

- 성능에 신경 써야한다면 많은 문자열을 연결할 때는 문자열 연결 연산자(+)를 피하자.
- 대신에 StringBuilder의 append 메서드를 사용하자.