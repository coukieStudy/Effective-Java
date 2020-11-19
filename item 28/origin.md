# Item 28. 배열보다는 리스트를 사용하라

배열과 제네릭 타입에는 중요한 차이가 두 가지 있다.
첫 번째, 배열은 공변 (covariant; 共變)이다. Sub가 Super의 하위 타입이라면 배열 Sub[]는 배열 Super[]의 하위 타입이 된다(공변, 즉 함께 변한다는 뜻이다).
반면, 제네릭은 불공변(invariant; 不共變)이다. 즉, 서로 다 른 타입 Type1과 Type2가 있을 때, List<Type1>은 List<Type2>의 하위 타입도 아니고 상위 타입도 아니다.

**예시**

런타임에 실패한다.
```java
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다."; // ArrayStoreException을 던진다.
```

컴파일되지 않는다!
```java
List<Object> ol = new ArrayList<Long>(); // 호환되지 않는 타입이다.
ol.add("타입이 달라 넣을 수 없다.");
```

두 번째 주요 차이로, 배열은 실체화(reify)된다.
다시말해, 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다.
반면, 제네릭은 타입 정보가 런타임에는 소거 (erasure)된다.

**이상의 주요 차이로 인해 배열과 제네릭은 잘 어우러지지 못한다.**
제네릭 배열을 만들지 못하게 막은 이유는 타입 안전하지 않기 때문
new List<E>[], new List<String>[], new E[] 식으로 작성하면 컴파일 오류.

제네릭 배열 생성을 허용하지 않는 이유
```java
List<String>[] stringLists = new List<String>[1];
List<Integer> intList = List.of(42);
Object[] objects = stringLists;
objects[0] = intList;  // 제네릭은 소거 방식으로 구현되어서 이 역시 성공한다.
String s = stringLists[0].get(0);
```

배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경 우 대부분은 배열인 E[] 대신 컬렉션인 List<E>를 사용하면 해결된다.
코드가 조금 복잡해지고 성능이 살짝 나빠질 수도 있지만, 그 대신 타입 안전성과 상 호운용성은 좋아진다.
  
```java  
public class Chooser {
  private final Object[] choiceArray;
  
  public Chooser(Collection choices) {
    choiceArray = choices.toArray();
  }
  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
```
```java  
public class Chooser<T> {
  private final T[] choiceArray;
  
  public Chooser(Collection<T> choices) {
    choiceArray = choices.toArray();
  }
  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
```
제네릭 사용
```java  
public class Chooser<T> {
  private final T[] choiceArray;
  
  public Chooser(Collection<T> choices) {
    choiceArray = (T[])choices.toArray();
  }
  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
```
경고 발생
```java  
public class Chooser<T> {
  private final List<T> choiceArray;
  
  public Chooser(Collection<T> choices) {
    choiceList = new ArrayList<>(choices);
  }
  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceList.get(rnd.nextInt(choiceList.size()));
  }
}
```
