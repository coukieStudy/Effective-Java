# Chapter 55 옵셔널 반환은 신중히 하라

## 자바 8 이전 값을 반환할 수 없는 경우, 선택지

1. 예외를 던진다
    - 진짜 예외상황(?)일 때만 사용해야 한다. → Item 69
    - 예외를 생성할 때 스택 추적 전체를 캡처하므로 비용도 만만치 않다.
2. null 반환
    - 사용하는 쪽에서 null check 로직이 들어가야 함.
    - NPE 발생 가능

## 자바 8에서의 새로운 선택지

- Optional<T> 사용: T타입 참조를 담거나, 아무것도 담지 않을 수 있다.
- 아래 예시는 Optional로 변환한 예시이다.
    - 주의할 점: 반환할 때 Optional.of()에 null을 넣으면 NPE. Optional.ofNullable()을 사용하면 안나긴 하지만, 사용하는 곳에서 get할 때 NPE 발생.

    ```java
    Optional<> a = max(...);
    a.get() //NPE 발생
    ```

```java
//Optional 사용 X
public static <E extends Comparable<E>> E max(Collection<E> c) {
  if (c.isEmpty()) {
    throw new IllegalArgumentException("빈 컬렉션");
  }
  E result = null;
  for (E e : c) {
    if (result == null || e.compareTo(result) > 0) {
      result = Objects.requireNonNull(e);
    }
  }
  return result;
}

//Optional 사용 O
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
  if (c.isEmpty()) {
    return Optional.empty();
  }
  E result = null;
  for (E e : c) {
    if (result == null || e.compareTo(result) > 0) {
      result = Objects.requireNonNull(e);
    }
  }
  return Optional.of(result);
}
```

### Optional 선택 기준

- 반환값이 없을 수도 있음을 API 사용자들에게 명확히 알려준다. orElseGet등으로 비어있을 때 처리하기도 편하다.
    - Null의 경우에도 같지 않나? → 항상 불안함에 떨어야 하는 null보다는 Optional이 괜찮다.
- filter, map, flatMap, ifPresent와 같이 사용 가능.

### isPresent 메서드

- 대부분의 isPresent 메서드는 orElseGet등으로 대체할 수 있다.
- 참고: [http://homoefficio.github.io/2019/10/03/Java-Optional-바르게-쓰기/](http://homoefficio.github.io/2019/10/03/Java-Optional-%EB%B0%94%EB%A5%B4%EA%B2%8C-%EC%93%B0%EA%B8%B0/)

```java
// 안 좋음
Optional<Member> member = ...;
if (member.isPresent()) {
    return member.get();
} else {
    return null;
}

// 좋음
Optional<Member> member = ...;
return member.orElseGet(null);
```

### orElse vs orElseGet

- orElseGet은 예외상황에만 invoke되기 때문에, orElse보다 효율적
- 참고: [https://www.baeldung.com/java-optional-or-else-vs-or-else-get](https://www.baeldung.com/java-optional-or-else-vs-or-else-get)

- orElse(): returns the value if present, otherwise return other
- orElseGet(): returns the value if present, otherwise **invoke** other and return the result of its invocation

### Stream<Optional<T>>

- Stream 내부에 채워진 Optional만 Stream<T>로 전달해서 처리하는 로직은 드물지 않다.

```java
//Java 8
streamOfOptionals
		.filter(Optional::isPresent)
		.map(Optional::get)

//Java 9 -> Optional::stream이 optional을 stream으로 변환해주는 메서드
streamOfOptionals
		.flatMap(Optional::stream)
```

### Optional을 사용하면 안되는 경우

- Collection, Stream, Array, Optional 같은 컨테이너 타입은 옵셔널로 감싸면 안된다.
    - ex) 빈 List를 반환하는 것이 더 좋다 → 옵셔널 처리 코드를 넣지 않아도 됨.
- Optional을 Map의 Key로 사용하면 안된다. 만약 사용한다면, key가 없는 경우는 두 가지이다. **→ 복잡**.
    1. 키가 Map에 존재하지 않는 경우
    2. 키가 Map에 있지만, Optional 내부가 빈 경우.
- Instance Field로 사용하는 경우
    - 애초에 Optional은 필드 사용으로 만들어진 것이 아니긴 하다.  Serializable도 구현하지 않았음.
    - Effective Java 말미에 사용하는 경우도 있다고 하는데... 안티 패턴인 거 같다.

### Optional을 사용하는 경우

- 결과가 없을 수도 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 Optional<T>를 반환한다.
- 단점
    - 항상 새로 할당해야 하기 때문에 성능이 민감하다면 좋지 않을 수 있다.

### Boxing 기본 타입을 담는 Optional

- 값을 두 겹이나 감싸기 때문에 더 무거울 수 밖에 없다. Boxing, UnBoxing
    - int → Integer → Optional<Integer>
- 이런 것들을 위해서 OptionalInt, OptionalLong, OptionalDouble을 제공한다.

```java
// 안 좋음
Optional<Integer> count = Optional.of(38);  // boxing 발생
for (int i = 0 ; i < count.get() ; i++) { ... }  // unboxing 발생

// 좋음
OptionalInt count = OptionalInt.of(38);  // boxing 발생 안 함
for (int i = 0 ; i < count.getAsInt() ; i++) { ... }  // unboxing 발생 안 함
```