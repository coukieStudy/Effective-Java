# Item 26. Raw Type은 사용하지말자!

## 용어 정리

- `Generic Class / Generic Interface` : 클래스 / 인터페이스 선언에 타입 매개변수가 쓰인 경우 (ex) List<E>
- `Generic Type` : Generic Class or Generic Infterface 
- `Parameterized Type` : 실제 타입 매개변수들이 나열된 경우 (ex) List<String>
- `Raw Type` : (ex) List

## Raw Type의 문제점과 존재 이유

**로타입을 사용하면 런타임에 에러가 발생할 수 있다.**

ex) List\<String\> Vs List

```java 
// 나중에 stamps.add(new Coin())을 수행할 경우

private final Collection stamps = ...; // 컴파일, 실행 가능
private final Conllection<Stamp> =  ...; // 컴파일 에러 발생
```

**왜 로타입은 존재하는 것인가?** 제네릭이 도입되기 이전 코드와의 호환성 때문.


## List\<Object\> Vs List


```java 
// 매개변수 list에 List<String>을 넘겨줄 경우

private static void unsafeAdd(List list, Object o) { 
  list.add(o);
}

private static void unsafeAdd(List<Object> list, Object o) { // 이 경우는 컴파일 에러 발생
  list.add(o);
}
```

raw Type은 타입 불변식을 훼손하기 쉽다.


## Unbounded wildcard type (비한정적 와일드카드 타입)

제네릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경쓰고 싶지 않다면 ?를 사용하자.

Collection<?>에는 (null 외에는) 어떤 원소도 넣을 수 없다. 따라서 이를 어기면 컴파일 에러 발생.

## 예외적으로 Raw Type 사용하는 경우

1. class 리터럴에는 Raw Type을 사용한다. 자바 명세는 class 리터럴에 매개변수화 타입을 사용하지 못하게 했다(배열과 기본타입은 허용한다).<br>
ex) List.class, String[].class (O)<br>
List<String>.class, List<?>.class (X)

2. instanceof는 로타입이든 비한정적 와일드 카드 타입이든 완전히 똑같이 동작하므로 차라리 로타입을 쓴다.
