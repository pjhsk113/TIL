# [이펙티브 자바] Item60 - 정확한 답이 필요하다면 float와 double은 피하라

float와 double 타입은 공학 계산용으로 설계되었다. 넓은 범위의 수를 빠르게, 정밀한 **근사치**로 계산하도록 설계되었다. 따라서 정확한 결과가 필요할 때는 사용하면 안 된다.

특히, 금융(돈) 관련 계산에는 float와 double 타입 사용을 피하자.

# 금융 계산에 부동소수 타입을 사용한 경우

```java
public static void main(String[] args) {
    double funds = 1.00;
    int itemBought = 0;
    for(double price = 0.10; funds >= price; price += 0.10) {
        funds -= price;
        itemsBought++;
    }
    
    System.out.println(itemBought + "개 구입");
    System.out.println("잔돈(달러): " + funds);
}
```

![](https://blog.kakaocdn.net/dn/cmTy3W/btra8A48evu/C5ZtTq463d3NQKpIxhSgzK/img.png)

잔돈으로 0.3999999999999가 출력된다. 당연히 우리가 원하는 결과는 아니다. 이처럼 정확한 계산이 필요한 금융 계산에 부동소수 타입을 사용하면 정확한 답이 아닌 **근사치**로 계산된다.

금융 계산에 정확한 결과를 원한다면 BigDecimal, int, long을 사용해야한다.

# BigDecimal보다는 int나 long을 사용하자

BigDecimal은 기본 타입보다 쓰기 불편하고, 훨씬 느리다.