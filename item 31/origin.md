# 한정적 와일드카드를 사용해 API 유연성을 높여라

### 한정적 와일드카드의 필요성
***
매개변수화 타입은 불공변(invariant)이다.

Type1과 Type2가 있을 때 List<Type1>과 List<Type2>는 상위/하위 타입도 아니다. 바로 그 점 때문에 유연성이 부족할 때도 있다.


```java
public void pushAll(Iterable<E> src) {
  for(E e : src)
    push(e);
}

Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ...;
numberStack.pushAll(integers);
```
위 코드는 컴파일 오류가 난다. Iterable<Integers> 가 Iterable<Number>에 대입될 수 없기 때문이다.

pushAll()의 입력 매개변수 타입은 'E의 iterable'이 아니라 'E의 하위 타입의 iterable'이어야 위의 상황이 해결되며, 이를 해결하기 위한 코드가
```java
pushAll(Iterable<? extends E>)
```
이다.
***
반대로 E의 하위타입이 아니라 E의 상위타입의 매개변수가 필요할 때도 있다.
```java
public void popAll(Collection<E> dst) {
  while(!isEmpty())
    dst.add(pop());
}
````

위의 코드에서 Stack<Number>에서 원소를 뽑아서 Collection<Objects>에 넣고 싶지만, 동일한 이유로 popAll()의 매개변수로 Collection<Objects>를 넣을 수 없다.
popAll()의 입력 매개변수 타입은 'E의 Collection'이 아니라 'E의 상위타입의 Collection'이어야 위의 상황이 해결되며, 이를 해결하기 위한 코드가
```java
popAll(Collection <? super E>)
```
이다.
***
위의 와일드카드 타입을 사용하면 유연성을 높일 수 있다. 상황별로 extends / super 를 써야하는데, 이를 위한 원칙은 다음과 같다.
> **PECS : producer-extends, consumer-super**

만약 한 인수가 producer / consumer 의 역할을 동시에 한다면 extends / super를 둘 다 쓰면 안된다.

두 Set를 union 하는 함수가 있을 때, 와일드카드를 쓰면 서로 다른 매개변수를 가진 Set을 Union할 수 있다. 
```java
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)

Set<Integer> integers = ...;
Set<Double> doubles = ...;
Set<Number> numbers = union(integers, doubles);
```
이 때 반환타입은 Set<E>를 써야한다. 반환타입에는 한정적 와일드카드를 사용하면 안된다. 클라이언트 코드에서도 반드시 한정적 와일드카드 타입을 써야하는 문제가 생긴다.
```java
public static List<? extends Integer> myList()

List<Integer> list = myList() // 컴파일 오류
List<? extends Integer> list = myList() // 이렇게 해야만 된다.
```

### 입력 매개변수와 타입 매개변수 모두에 한정적 와일드카드 적용
***
입력으로 주어진 List에서 가장 가장 큰 원소를 반환하는 함수.
```java
//원래 함수
public static <E extends Comparable<E>> E max1(List<E> list)
// 한정적 와일드카드 적용
public static <E extends Comparable<? super E>> E max2(List<? extends E> list) 
```
입력 매개변수에의 적용: 'E 의 List' 뿐만 아니라 'E의 하위타입의 List'도 parameter로 받을 수 있게 확장
타입 매개변수에의 적용: 원래는 Comparable 을 직접 구현한 class (E)만 타입 매개변수로 받을 수 있었지만, Comparable을 직접 구현하지 않은 클래스더라도, 상위 클래스에서 Comparable을 구현한 클래스도 타입 매개변수가 될 수 있다.

```java
// MyNumber는 Comparable 을 구현했지만 하위클래스인 MyInteger는 구현하지 않은 상태

class MyNumber implements Comparable<MyNumber> {
  public Number number;
  public MyNumber(Number number) {this.number = number);
  public int compareTo(MyNumber o) {return 0};
}

class MyInteger extends MyNumber {
  public Integer integer;
  public MyInteger(Integer integer) {super(integer); this.integer = integer};
}

...
List<MyInteger> myIntegers = ...;
max1(myIntegers); // 컴파일 실패
max2(myIntegers); // 성공
```
MyInteger의 인스턴스는 다른 MyInteger객체 뿐 아니라 MyNumber 인스턴스와도 비교할 수 있어서 max2만 성공한다.

```java
public static <E extends Comparable<E>> E value1(E e) {return e};
public static <E extends Comparable<? super E>> E value2(E e) {return e};

...
// 다음 중 실패하는 것은?
MyNumber one = value(new MyNumber(1));
MyNumber two = value(new MyInteger(2));
MyInteger three = value(new MyInteger(3));
MyInteger four = newValue(new MyInteger(4));
```

***
### 타입매개변수와 와일드카드 사이에서의 선택
```java
public static <E> void swap(List<E> list, int i, int j);
public static void(List<?> list, int i, int j);
```

일반적으로는 간단한 두 번째 API가 낫다. 신경 써야 할 타입 매개변수도 없다.
그렇지만 구현을 위해서는 제네릭을 사용한 private 도우미 메서드를 사용해야 할 수도 있다.
```java
public static <E> void swapHelper(List<E> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));
}
