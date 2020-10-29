# Chapter 21 인터페이스는 구현하는 쪽을 생각해 설계하라

## 인터페이스에 메서드를 추가하는 방법

 - 인터페이스에 메서드를 추가하면 보통은 컴파일 오류가 나 는데, 추가된 메서드가 우연히 기존 구현체에 이미 존재할 가능성은 아주 낮기 때문이다.
 - 디폴트 메서드는 구현 클래스에 대해 아무것도 모른 채 합의 없이 무작정 ‘삽입’될 뿐이다. 하지만 생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하기란 어려운 법이다.

```java
default boolean removeIf(Predicate<? super E> filter) {
  Objects.requireNonNull(filter);
  boolean result = false;
  for (Iterator<E> it = iterator(); it.hasNext(); ) {
    if (filter.test(it.next())) {
      it.remove();
      result = true;
    }
  }
  return result;
}
```

 - org. apache.commons.collections4.collection.SynchronizedCollection와 충돌한다.
 
모든 메서드에서 주어진 락 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼 클래스(?)
removeIf의 구현은 동기화에 관해 아무것 도 모르므로 락 객체를 사용할 수 없다.

## 인터페이스를 설계 할 때는 여전히 세심한 주의를 기울여야 한다.
