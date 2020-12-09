## 람다보다는 참조메서드를 사용하라

다음 코드를 보자.
```java
map.merge(key, 1, (count, incr) -> count + incr);

//참고용 Map.java 
default V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction)
    Objects.requireNonNull(remappingFunction);
    Objects.requireNonNull(value); 
    V oldValue = get(key)
    V newValue = (oldValue == null) ? value :
           remappingFunction.apply(oldValue, value);
    if(newValue == null) {
        remove(key);
    } else {
        put(key, newValue);
    }
    return newValue;
}

```

Integer.sum을 이용하는 메서드 참조를 이용하면 다음과 같이 쓸 수 있다.

```java
map.merge(key, 1, Integer::sum);
```

람다와 동일한 기능을 하면서도 메서드를 이용한 naming 이 가능하기 때문에 많은 경우에 메서드 참조를 이용하는 것이 가독성이 더 좋다. 그러나 클래스 이름이 지나치게 길어서 메서드 참조가 오히려 가독성을 떨어트리는 경우나, 람다를 쓰는 것이 더 명확한 경우에는 람다를 쓰는 편이 낫다.
예를들어 Function.identity() 보다는 같은 역할을 하는 람다 (x -> x)가 더 명확하므로 이 경우에는 람다를 쓰는 것이 좋다.

메서드 참조의 유형

|<center>메서드 참조 유형</center> |<center>예시</center> |<center>같은 기능을 하는 람다</center> |
|--------|----------------------|--------------------------|
|정적|Integer::parseInt|str -> Integer.parseInt(str)|
|한정적 인스턴스|Instant.now()::isAfter|Instant then = Instant.now();<br>t -> then.isAfter(t)|
|비한정적 인스턴스|String::toLowerCase|str -> str.lowerCase()|
|클래스 생성자|```TreeMap<K, V>::new```|```() -> new TreeMap<K, V>()```|
|배열 생성자|int[]::new|len -> new int[len]|

참고) 람다로는 불가능하나 메서드 참조로는 가능한 유일한 경우는 제네릭 함수 타입의 구현이다.
```java
interface G1 {
	<E extends Exception> Object m() throws E;
}
interface G2 {
	<F extends Exception> String m() throws Exception; }
  
interface G extends G1, G2 {}
```

이를 함수형으로 바꾸면
```java
<F extends Exception> ()->String throws F
```
가 된다. 이처럼 함수형 인터페이스를 위한 제네릭 함수 타입은 메서드 참조 표현식으로는 구현할 수 있지만, 람다식으로는 불가능하다. 제네릭 람다식이라는 문법이 존재하지 않기 때문이다.
