# Chapter 54 null이 아닌, 빈 컬렉션이나 배열을 반환하라

- 아래처럼 null을 반환한다면, client는 이 null 상황을 처리하는 코드를 추가로 작성해야 한다.
    - null check 로직이 없다면, NPE 발생 가능.

```java
private final List<Cheese> cheesesInStock = ...;
/**
 * @return 매장 안의 모든 치즈 목록을 반환한다. * 단, 재고가 하나도 없다면 null을 반환한다. */
public List<Cheese> getCheeses () {
  return cheesesInStock.isEmpty() ? null: new ArrayList<>(cheesesInStock);
}
```

```java
//호출하는 쪽에서 null check를 아래와 같이 매번 해줘야 한다.
List<Cheese> result = getCheeses();
if (Objects.nonNull(result) && ...)
```

- 빈 컨테이너 반환이 null 반환보다 좋다. 보통 아래와 같이 사용.

    ```java
    public List<Cheese> getCheeses() {
      return new ArrayList<>(cheesesInStock); 
    }
    ```

- 성능을 더 높일 수 있는 방법: 빈 불변 컬렉션을 반환
    - Collections.emptyList()
    - Collections.emptySet()
    - Collections.emptyMap()

```java
//emptyList 사용
public List<Cheese> getCheeses () {
  return cheesesInStock.isEmpty() ? Collections.emptyList()
      : new ArrayList<>(cheesesInStock);
}

//Collections에 EMPTY_LISTS 구현
public static final List EMPTY_LIST = new EmptyList<>();

public static final <T> List<T> emptyList() {
    return (List<T>) EMPTY_LIST;
}
```

- 배열의 경우도 동일하다.

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
  return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

### 하지만 기본 프로젝트에서는 null을 자주 사용하는 경우도 있으니, 잘 보고 결정하자..