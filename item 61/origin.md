# 박싱된 기본 타입보다는 기본 타입을 사용하라

## 기본 타입과 박싱된 타입의 차이
1. 박싱된 기본 타입은 값과 함께 식별성이란 속성을 가진다.
    - 그러므로 박싱된 기본 타입의 두 인스턴스는 값이 같아도 다를 수 있다.
2. 박싱된 기본 타입은 null일 수 있다.
3. 기본 타입이 시간적, 공간적으로 더 효율적이다.

## 예시: Integer 비교자
```java
Comparator<Integer> naturalOrder =
    (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
```
- 일반적으로는 문제가 없는 비교자이지만...
- `naturalOrder.compare(new Integer(42), new Integer(42))`의 결과가 0이 아닌 1을 출력한다.
- 왜? `==`의 평가에서 두 객체 참조의 식별성을 검사하기 때문이다.
- 실무에서는 `Comparator.naturalOrder()`를 사용하자.
- 굳이 직접 구현하려면 이렇게 번거롭다.
```java
Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> {
    int i = iBoxed, j = jBoxed; // Auto-unboxing
    return i < j ? -1 : (i == j ? 0 : 1);
};
```

## 예시: Unbelievable!
```java
public class Unbelievable {
    static Integer i;

    public static void main(String[] args) {
        if (i == 42)
            System.out.println("Unbelievable");
    }
}
```
- 이 프로그램은 비교를 수행하기도 전에 NPE를 뱉는다.
- 박싱된 기본 타입과 기본 타입을 비교할 때 박싱된 기본 타입의 박싱이 풀리고 (`i.intValue()`), 이때 `i`의 값이 `null`이므로 NPE가 발생한다.
- `i`를 `int`로 바꿔주면 해결된다.

## 예시: 엄청 느린 계산
```java
public static void main(String[] args) {
    Long sum = 0L;
    for (long i = 0; i <= integer.MAX_VALUE; i++) {
        sum += i;
    }
    System.out.println(sum);
}
```
- 이 프로그램은 매 for loop마다 박싱과 언박싱을 반복하기 때문에 성능이 매우 구리다.

## 언제 박싱된 기본 타입을 써야 하는가?
1. 컬렉션의 원소, 키, 값으로 쓸 때
2. 매개변수화 타입으로 쓸 때
3. 리플렉션을 통해 메서드를 호출할 때 (?)
    - https://stackoverflow.com/questions/53908254/how-to-use-reflection-to-invoke-a-method-with-primitive-arguments (아닐 수도 있음)