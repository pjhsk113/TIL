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

![%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20Item60%20-%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%92%E1%85%AA%E1%86%A8%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%83%E1%85%A1%E1%86%B8%E1%84%8B%E1%85%B5%20%E1%84%91%E1%85%B5%E1%86%AF%E1%84%8B%E1%85%AD%E1%84%92%E1%85%A1%E1%84%83%20758ce7310cde44f3ad87ba458b6da844/Untitled.png](%5B%E1%84%8B%E1%85%B5%E1%84%91%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%5D%20Item60%20-%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%92%E1%85%AA%E1%86%A8%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%83%E1%85%A1%E1%86%B8%E1%84%8B%E1%85%B5%20%E1%84%91%E1%85%B5%E1%86%AF%E1%84%8B%E1%85%AD%E1%84%92%E1%85%A1%E1%84%83%20758ce7310cde44f3ad87ba458b6da844/Untitled.png)

잔돈으로 0.3999999999999가 출력된다. 당연히 우리가 원하는 결과는 아니다. 이처럼 정확한 계산이 필요한 금융 계산에 부동소수 타입을 사용하면 정확한 답이 아닌 **근사치**로 계산된다.

금융 계산에 정확한 결과를 원한다면 BigDecimal, int, long을 사용해야한다.

# BigDecimal보다는 int나 long을 사용하자

BigDecimal은 기본 타입보다 쓰기 불편하고, 훨씬 느리다.