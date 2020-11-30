# Chapter 30 이왕이면 제너릭 메서드로 만들라.

## 로 타입은 타입 안전하지 않기 때문에 사용하지 말자.

```java
//unchecked 에러 발생
public static Set union(Set s1, Set s2) {
  Set result = new HashSet(s1);
  result.addAll(s2);
  return result;
}
```

## 타입 안전한 코드 - Generic 사용

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
  Set<E> result = new HashSet<>(s1);
  result.addAll(s2);
  return result;
}
```

- 위 코드는 파라미터 2개와 return 1개의 타입이 모두 같아야 한다. 이를 bounded 타입을 사용하여 더 유연하게 개선할 수 있다.

## Generic

- Generic의 장점: **컴파일 타임**에 타입 안전성을 보장받을 수 있어, UncheckedException과 ClassCastException에서 안전할 수 있다.

### Generic Type Erasure(타입 소거)

- 컴파일 타임에는 타입을 검사하고, 런타임에는 해당 타입 정보를 알 수 없는 것.
- Java 5에서 타입 소거가 적용될 때는 하위 버전을 지원하고자 만들었지만, 아래의 장점이 존재한다.
- 장점
    - 가독성이 상승
    - 추가적인 정보가 없음으로 성능이 올라간다.

## Generic Singleton Factory

- 항등함수(identity function)를 담은 클래스를 만들고 싶다.
- 간단한 방법: Function.identity
    - Function<T, R>로서 받은 매개변수를 그대로 return한다.

```java
final Function<Integer, Integer> identity = Function.identity();
System.out.println(identity.apply(10000));
```

- 공부하기 위해 구현해보면

```java
public class GenericSingletonFactory {
    // Generic singleton factory pattern
    private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

    @SuppressWarnings("unchecked")
    public static <T> UnaryOperator<T> identityFunction() {
        return (UnaryOperator<T>) IDENTITY_FN; //안전
    }

    // Sample program to exercise generic singleton
    public static void main(String[] args) {
        String[] strings = { "jute", "hemp", "nylon" };
        UnaryOperator<String> sameString = identityFunction();
        for (String s : strings)
            System.out.println(sameString.apply(s));

        Number[] numbers = { 1, 2.0, 3L };
        UnaryOperator<Number> sameNumber = identityFunction();
        for (Number n : numbers)
            System.out.println(sameNumber.apply(n));
    }
}
```

## 재귀적 타입 한정

- '모든 타입 E는 자신과 비교할 수 있다' 라는 뜻. compareTo를 구현한 것들

```java
<E extends Comparable<E>>
```

- 재귀적 타입 한정이 생긴 이유는 연산자 오버로딩을 지원하지 않는 Java 언어의 한계 때문이다.

```java
public static <T> int countGreaterThan(T[] anArray, T elem) {
    int count = 0;
    for (T e : anArray)
        if (e > elem)  // compiler error
            ++count;
    return count;
}

public static <T extends Comparable<T>> int countGreaterThan(T[] anArray, T elem) {
    int count = 0;
    for (T e : anArray)
        if (e.compareTo(elem) > 0)
            ++count;
    return count;
}
```
