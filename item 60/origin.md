# 정확한 답이 필요하다면 float과 double은 피하라

- `float`과 `double`은 과학 및 공학 계산용으로 설계되었다. 따라서 정확도보단 속도가 중요하기 때문에 근사치를 계산한다.
- 특히 10의 음의 제곱수를 표현할 수 없기 때문에 금융 관련 계산에 써서는 안된다.
- 부정확한 이진 부동소수점 계산의 예시
    - `1.03 - 0.42 = 0.6100000000001`
    - `1.00 - 9 * 0.10 = 0.09999999999998`
- 반올림하면 되지 않음?
    - 1달러가 있고, 사탕 10센트, 20센트, 30센트, ... 1달러짜리가 있다고 했을 때 사탕을 몇 개까지 살 수 있고 잔돈이 얼마나 남는지 구하는 코드
    ```java
    public static void main(String[] args) {
        double funds = 1.00;
        int itemsBought = 0;
        for (double price = 0.10; funds >= price; price += 0.10) {
            funds -= price;
            itemsBought++;
        }
        System.out.println(itemsBought + " items bought.");
        System.out.println("Change: $" + funds);
    }
    ```
    - 결과는 3개, 0.399999999달러가 나옴. 마지막 값을 반올림해도 틀린 답임.
    - 매 for loop 마다 반올림하면 되지 않음? -> 인생이 너무 고달파짐
- 답은 `BigDecimal`이다.
```java
public static void main(String[] args) {
    // 오차를 방지하기 위해 문자열을 받는 생성자를 사용했다.
    final BigDecimal TEN_CENTS = new BigDecimal(".10");

    int itemsBought = 0;
    BigDecimal funds = new BigDecimal("1.00");
    for (BigDecimal price = TEN_CENTS;
            funds.compareTo(price) >= 0;
            price = price.add(TEN_CENTS)) {
        funds = funds.subtract(price);
        itemsBought++;
    }
    System.out.println(itemsBought + " items bought.");
    System.out.println("Money left over: $" + funds);
}
```
- 이렇게 하면 올바른 답이 나온다: 사탕 4개, 잔돈 0달러.
- `BigDecimal`의 단점: 쓰기 불편하다, 느리다.
- 대안: `int`나 `long`을 사용하자. 위 예시에선 달러를 센트 단위로 바꾸면 될 것이다.