# 제네릭과 가변인수를 함께 쓸 때는 신중하라
- 가변인수란? 메서드에 임의의 개수의 인수를 입력할 수 있도록 허용하는 것.
```java
public static void main(String... args)
```
- 가변인수 메서드를 호출하면 가변인수를 담은 배열이 하나 만들어지는데, 이 배열을 클라이언트에게 노출하는 허점이 존재한다. 노출한다는게 뭔 소리냐? 클라이언트가 가변인수 메서드를 오버라이딩하여 배열을 마음대로 사용할 수 있다.
```java
@Override
public static String[] expose(String... args) {
    // args를 마음대로 사용
}
```
- 그리고 아이템 28에서 배웠듯이 제네릭과 배열은 잘 어울리지 못한다. 그러므로 제네릭을 가변인수로 쓸 때도 신중해야 한다. 다만 차이점은 제네릭을 배열로 선언하는 건 컴파일 타임에 막히지만, 제네릭을 가변인수로 선언하는 건 컴파일 타임에 경고만 받고 넘어간다.
- 그 결과 다음과 같은 코드가 힙 오염을 발생시킬 수 있다.
```java
static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intList; // 여기서 힙 오염 발생
    String s = stringLists[0].get(0); // ClassCastException
}
```
- 그러므로 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.
- 그럼 이걸 왜 컴파일 허용할까? 너무 유용해서. (아이템 31의 항등함수 예시와 비슷하다)
- 실제로 자바 라이브러리도 이런 메서드들을 제공한다. `Arrays.asList(T... a)`, `Collections.addAll(Collection<? super T>)`, `EnumSet.of(E first, E... rest)` 등.
- 위 메서드들은 타입 안전하다. (static이라 오버라이딩 불가)

## @SafeVarargs 애너테이션을 사용하자
- @SafeVarargs는 자바 7에서 추가되었으며 해당 가변인수 메서드가 타입 안전하다고 알려줌으로써 경고를 숨기는 역할을 한다.
- 가변인수 메서드가 타입 안전한 조건
    - varargs 배열에 아무것도 저장하지 않는다 (ex. `varargs[0] = something;`).
    - varargs 배열의 참조를 밖으로 노출시키지 않는다 (ex. `return varargs;`).
- 타입 안전하지 않은 가변인수 메서드의 예
```java
static <T> T[] toArray(T... args) {
    // 컴파일러는 args를 담기 위해 Object[] 배열을 생성한다
    // 그래야 어떤 타입이든 담을 수 있기 때문이다.
    return args;
    // 리턴 타입은 Object[] 이다.
}

static <T> t[] pickTwo(T a, T b, T c) {
    switch(ThreadLocalRandom.current().nextInt(3)) {
        case 0: return toArray(a, b);
        case 1: return toArray(a, c);
        case 2: return toArray(b, c);
    }
    throw new AssertionError();
    // 위와 마찬가지로 리턴 타입은 Object[] 이다.
}

public static void main(String[] args) {
    String[] attributes = pickTwo("좋은", "빠른", "저렴한")
    Object[] 를 String[]로 형변환 시도하여 예외가 발생한다.
}
```
- 즉 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다.
- 두 가지 예외
    - @SafeVarargs 애너테이션이 있는 다른 가변인수 메서드에 넘기는 경우
    - 변수에 할당되지 않고 호출만 될 경우
- 타입 안전한 가변인수 메서드의 예
```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
        result.addAll(list);
    return result;
}
```
- 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드를 안전하게 만들고 @SafeVarargs 애너테이션을 달아야 한다.
- @SafeVarargs는 static 혹은 final 메서드에만 붙일 수 있다 (자바 9부터는 private 메서드에도).

## 대안: varargs 매개변수를 List 매개변수로 바꾸기
위의 `flatten`을 다음과 같이 변경할 수 있다.
```java
static <T> List<T> flatten(List<List<? extends T>> lists) {
    // 아래 내용은 동일하다.
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
        result.addAll(list);
    return result;
}
```
- 클라이언트 쪽에서는 `List.of`를 사용해 변경된 `flatten`에 임의 개수의 인수를 넘길 수 있다. `List.of`에는 @SafeVarargs가 달려 있기 때문에 이것이 가능하다 (자바 9 이상만 가능).
```java
audience = flatten(List.of(friends, romans, countryman)); // 자바 9 이상
audience = flatten(Arrays.asList(friends, romans, countryman)); // 자바 8
```
- 장점
    - @SafeVarargs를 직접 달지 않아도 된다.
    - 가변인수 메서드를 안전하게 사용하기 불가능한 상황에서도 쓸 수 있다.
    ``` java
    // 리턴 타입을 바꿔준다.
    static <T> List<T> pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return List.of(a, b);
            case 1: return List.of(a, c);
            case 2: return List.of(b, c);
        }
        throw new AssertionError();
    }

    public static void main(String[] args) {
        List<String> attributes = pickTwo("좋은", "빠른", "저렴한");
    }
    ```
- 단점
    - 클라이언트 코드가 지저분해진다.
    - 속도가 조금 느려진다.