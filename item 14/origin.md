# Comparable을 구현할지 고려하라
- 다른 메서드들과 달리 `compareTo`는 `Object`가 아닌 `Comparable` 인터페이스의 메서드이다.
- `compareTo`는 동치성 비교와 더불어 순서까지 비교할 수 있다. 즉 `Comparable`의 구현체에는 자연적 순서가 있다.
- `Comparable`의 구현체는 다음과 같이 활용할 수 있다.
```java
// Comparable 구현체의 배열을 정렬할 수 있다
Arrays.sort(a);

// 자동 정렬되는 컬렉션에 넣을 수 있다
Set<String> s = new TreeSet<>();
Collections.addAll(a);
```
- 그러므로 순서가 명확한 값 클래스를 작성한다면 반드시 `Comparable`을 구현하자.

## compareTo 메서드의 일반 규약
- `compareTo` 메서드의 일반 규약은 `equals`와 비슷하다. 내용은 다음과 같다.
	- `x.compareTo(y)`는 `x`와 `y`의 순서를 비교한다. `x`가 `y`보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다.
	- `y`가 `x`와 비교할 수 없는 타입이면 `ClassCastException`을 던진다.
	- 모든 `x`, `y`에 대해 `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))` 여야 한다. (`sgn`은 부호 함수)
	- `(x.compareTo(y) > 0 && y.compareTo(z) > 0` 이면 `x.compareTo(z) > 0` 이다.
	- 모든 `z`에 대해 `x.compareTo(y) == 0`이면 `sgn(x.compareTo(z)) == sgn(y.compareTo(z))`이다.
	- 권고: `(x.compareTo(y) == 0) == (x.equals(y))`여야 한다. 그렇지 않을 경우 주석에 그 사실을 명시해야 한다.
- 이 규약을 지키지 않으면 `TreeSet`, `TreeMap`, `Collections`, `Arrays`와 같이 비교를 활용하는 클래스와 어울리지 못한다.
- `equals`와 마찬가지로 하위 클래스에서 새로운 값 컴포넌트를 추가한 경우엔 이 규약을 지킬 수 없으며, 우회법도 같다 (상속 대신 컴포지션 사용).
- 마지막 규약을 지키지 않을 경우, 이 클래스의 객체를 정렬된 컬렉션에 넣을 때 의도하지 않게 동작할 수 있다. 정렬된 컬렉션들은 동치성을 비교할 때 `equals`,가 아닌 `compareTo`를 사용하기 때문이다.
```java
BigDecimal decimal1 = new BigDecimal("1.0");
BigDecimal decimal2 = new BigDecimal("1.00");
Set a = new HashSet<>();
a.add(decimal1);
a.add(decimal2);
// a의 원소는 2개이다. decimal1.equals(decimal2)가 false이기 때문이다.

Set b = new TreeSet<>();
b.add(decimal1);
b.add(decimal2);
// b의 원소는 1개이다. decimal1.compareTo(decimal2)가 0이기 때문이다.
```
- 실제 `BigDecimal`의 `equals`의 주석의 일부는 다음과 같다.
	- Unlike {@link  #compareTo(BigDecimal) compareTo}, this method considers two {@code BigDecimal} objects equal only if they are equal in value and scale (thus 2.0 is not equal to 2.00 when compared by this method).

## compareTo 메서드 작성 요령
- `Comparable`은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo 메서드의 인수 타입은 컴파일 타임에 정해진다. 그러므로 입력 인수의 타입을 확인할 필요가 없다.
```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```
- 객체 참조 필드를 비교하려면 `compareTo` 메서드를 재귀적으로 호출한다.
- 다음과 같은 경우에는 비교자를 사용한다.
	- `Comparable`을 구현하지 않은 필드를 비교할 때
	- 표준이 아닌 순서로 비교해야 할 때 (여기서 "표준"이란 `compareTo` 메서드를 말한다)
	- 정수 기본 타입 혹은 실수 기본 타입 필드를 비교할 때 (각각의 래퍼 클래스에서 제공하는 `compare`를 사용한다)	
- 핵심 필드가 여러 개라면 가장 핵심 필드부터 비교한다. 순서가 결정될 때까지 모든 필드를 비교한다.
```java
public int compareTo(PhoneNumber pn) {
	int result = Short.compare(areaCode, pn.areaCode);
	if (result == 0) {
		result = Short.compare(prefix, pn.prefix);
	}
	if (result == 0) {
		result = Short.compare(lineNum, pn.lineNum);
	}
	return result;
}
```
- 위 예시는 `Comparator` 인터페이스를 이용해 메서드 체이닝을 적용할 수 있다.

## Comparator 인터페이스
- `Comparator` 인터페이스를 이해하지 못하면 이 장의 나머지 내용이 헷갈릴 수 있기 때문에 책보다 좀 더 깊이 다루겠습니다.
- `Comparator`를 구현하는 클래스는 `compare`와 `equals` 메서드를 구현해야 한다.
```java
@FunctionalInterface
public interface Comparator<T> {

	int compare(T o1, T o2);

	boolean equals(Object obj);
}
```
- `compare(T o1, T o2)`는 `o1`이 `o2`보다 작을 때, 같을 때, 클 때 각각 음의 정수, 0, 양의 정수를 반환한다.
- `equals(Object obj)`는 `obj`가 `Comparator` 객체이며 이 객체와 같은 비교자를 제공하는지 여부를 반환한다 (중요하지 않음).
- 인터페이스라는 점이 무색하게 `Comparator`는 위 두 메서드 외에 7개의 디폴드 메서드와 9개의 정적 메서드를 가지고 있다. 총 16개 메서드 모두 반환 타입이 `Comparator`이다. 이것이 의미하는 건 이 16개 메서드들이 모두 메서드 체이닝에 사용된다는 것이다.
- 가장 중요한 `comparing` 메서드는 다음과 같다.
```java
public static <T, U> Comparator<T> comparing(  
        Function<? super T, ? extends U> keyExtractor,  
		Comparator<? super U> keyComparator) {  
    Objects.requireNonNull(keyExtractor);  
	Objects.requireNonNull(keyComparator);  
	return (Comparator<T> & Serializable)  
        (c1, c2) -> keyComparator.compare(keyExtractor.apply(c1),  
										  keyExtractor.apply(c2));  
}
```
- 여기서 `keyExtractor`는 비교에 쓰일 객체 key를 추출하는 함수이며, 추출되는 객체는 `keyComparator`의 `compare`의 인자로 쓸 수 있는 것이어야 한다.
- `keyComparator`는 비교자를 제공하는 `Comparator` 객체이다.
- 언제 이 메서드를 쓸 수 있을까? 만약 어떤 타입 `T`인 객체 `t1`과 `t2`를 비교할 수 있는 비교자를 만들고 싶은데, `T`의 필드 중 하나인 타입 `U`인 필드 `u`를 가지고 비교하고 싶다고 해보자. 그리고 타입 `U`인 객체끼리 비교할 수 있는 `Comparator` 구현체 `comparatorOfU`가 있다고 해보자. 그러면 다음과 같이 구현한 후 언제든지 재사용할 수 있다.
```java
Comparator<T> comparatorOfT = Comparator.comparing(
	(T t) -> t.getU(), comparatorOfU
);
```
- 만약 `U`가 `Comparable`의 구현체라면 다음과 같이 쓸 수 있다.
```java
Comparator<T> comparatorOfT = Comparator.comparing(
	(T t) -> t.getU(),
	(u1, u2) -> return u1.compareTo(u2)
);
```
- `Comparator`는 인자가 `keyExtractor` 하나뿐인 `comparing` 메서드도 제공하므로 다음과 같이 더 간단히 쓸 수도 있다.
```java
Comparator<T> comparatorOfT = Comparator.comparing((T t) -> t.getU());
```
- 위와 같이 `comparing` 메서드를 사용해 `Comparator` 객체를 만든 후에는 `thenComparing` 메서드를 체이닝해 비교 로직을 추가할 수 있다. `thenComparing`의 코드는 다음과 같다.
```java
default Comparator<T> thenComparing(Comparator<? super T> other) {  
    Objects.requireNonNull(other);  
	return (Comparator<T> & Serializable) (c1, c2) -> {  
        int res = compare(c1, c2);  
		return (res != 0) ? res : other.compare(c1, c2);  
	};  
}

default <U> Comparator<T> thenComparing(  
    Function<? super T, ? extends U> keyExtractor,  
	Comparator<? super U> keyComparator) {  
    return thenComparing(comparing(keyExtractor, keyComparator));  
}

default <U extends Comparable<? super U>> Comparator<T> thenComparing(  
    Function<? super T, ? extends U> keyExtractor) {  
    return thenComparing(comparing(keyExtractor));  
}
```
- 즉 앞의 비교자에 의한 비교가 0이면 `thenComparing`을 체이닝하여 제공한 비교자로 비교한 결과를 반환하도록 할 수 있다.
- 만약 위 예시에서 `T`의 필드 `u`로 비교한 결과가 0이라면 다른 필드인 타입 `V`인 필드 `v`로 비교하고 싶다고 해보자 (`V`는 `Comparable`의 구현체라 가정하자). 그러면 다음과 같이 구현하면 된다.
```java
Comparator<T> comparatorOfT = Comparator.comparing((T t) -> t.getU())
	.thenComparing(t -> t.getV());
```
- `Comparator` 인터페이스에는 `comparingInt`, `comparingLong`, `comparingDouble`, `thenComparingInt`, `thenComparingLong`, `thenComparingDouble`도 있다. 모두 각각의 이름에 해당하는 기본 타입의 값을 추출해 비교하는 비교자를 만들고 싶을 때 쓰는 메서드들이다. 그러므로 위의 `PhoneNumber` 클래스의 예시는 다음과 같이 쓸 수 있다.
```java
private static final Comparator<PhoneNumber> COMPARATOR =
	comparingInt((PhoneNumber pn) -> pn.areaCode)
		.thenComparingInt(pn -> pn.prefix)
		.thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
	return COMPARATOR.compare(this, pn);
}
```
- 이 방식은 그냥 `compareTo`만 구현하는 것보다 성능이 뒤떨어진다.
- 값의 차를 기준으로 비교를 수행하는 `Comparator` 구현체를 만들면 안 된다. 정수일 경우 정수 오버플로를 일으킬 수 있고, 부동소수점 자료형일 경우 실제로 같은 값을 작거나 크다고 판별할 수 있다.
```java
// 이렇게 구현하면 안 된다.
static Comparator<Object> hashCodeOrder = new Comparator<>() {
	public int compare(Object o1, Object o2) {
		return o1.hashCode() - o2.hashCode();
	}
};

// 이렇게 구현하거나
static Comparator<Object> hashCodeOrder = new Comparator<>() {
	public int compare(Object o1, Object o2) {
		return Integer.compare(o1.hashCode(), o2.hashCode());
	}
};

// 이렇게 구현하자
static Comparator<Object> hashCodeOrder = 
	Comparator.comparingInt(o -> o.hashCode());
```