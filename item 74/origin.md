# Chapter 74 메서드가 던지는 모든 예외를 문서화하라

- 검사 예외는 항상 따로따로 선언하고, 각 예외가 발생하는 상황을 자바독의 @throws 태그를 사용하여 정확히 문서화하자
- 잘못된 방법

```java
public void method() throws Exception {
}
```

- 좋은 방법

```java
public void method() throws NullPointException... {
}
```

- 메서드가 던질 수 있는 예외를 각각 @throws 태그로 문서화하되, 비검사 예외는 메서드 선언의 throws 목록에 넣지 말자.
- 거의 모든 메서드에서 같은 예외를 던진다면 Class 설명에 추가하자.