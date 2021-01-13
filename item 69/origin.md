#### Item 69. 예외는 진짜 예외 상황에만 사용하라

운이 없다면 언제가 아래와 같은 코드와 마주칠지도 모른다.

```java
try {
	int i = 0;
	while(true) 
    range[i++].climb();
} catch (ArrayIndexOutOfBoundsException e) {
}
```

정상적인 코드

```java
for (Mountain m : range) 
  m.climb();
```

JVM은 배열에 접근할 때마다 경계를 넘지 않는지 검사한다는 점에서 위의 코드가 성능을 높일 수 있다고 착각할지 모르지만 JVM의 최적화를 방해할 뿐이다.

- **예외는 (그 이름이 말해주듯) 오직 예외 상황 에서만 써야 한다. 절대로 일상적인 제어 흐름용으로 쓰여선 안 된다.**

- **잘 설계된 API라면 클라이언트가 정상적 인 제어 흐름에서 예외를 사용할 일이 없게 해야 한다.** 

  특정 상태에서만 호출할 수 있는 ‘상태 의존적’ 메서드를 제공하는 클래스는 ‘상태 검사’ 메서드도 함께 제공해야 한다. Iterator 인터페이스의 next와 hasNext가 각각 상태 의존적 메서드와 상태 검사 메서드에 해당한다.

```java
for (Iterator<Foo> i = collection.iterator(); i.hasNext(); ) { 
  Foo foo = i.next();
	... 
}
```

**예외는 예외 상황에서 쓸 의도로 설계되었다. 정상적인 제어 흐름에서 사용해서는 안 되 며, 이를 프로그래머에게 강요하는 API를 만들어서도 안 된다.**
