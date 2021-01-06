# 문자열 연결은 느리니 주의하라

- 연산자 `+`로 문자열 n개를 잇는 시간은 O(n**2)이다.
- 그러니까 그럴 땐 `StringBuilder`를 사용해라.
```java
public String example() {
    StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
    for (int i = 0; i < numItems(); i++)
        b.append(lineForItem(i));
    return b.toString();
}
```