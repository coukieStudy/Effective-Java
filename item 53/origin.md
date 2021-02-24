# Chapter 53 가변인수는 신중히 사용하라

- 가변 인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다. 아래와 같이 sum의 경우는 인수가 몇 개든 상관없다.

```java
static int sum(int... args) { 
	int sum = 0;
	for (int arg : args) 
		sum += arg;
	return sum; 
}
```

- 하지만 인수가 1개 이상이어야 하는 경우도 있다.
    - 최솟값을 찾는 경우, 1개 이상의 인수를 받는 것이 좋다.

```java
//좋지 못한 구현 방법: 런타임에 배열의 길이를 알 수 있기에
static int min(int... args) { 
	if (args.length == 0)
		throw new IllegalArgumentException("인수가 1개 이상 필요합니다."); 
	int min = args[0];
	for (int i = 1; i < args.length; i++)
		if (args[i] < min) 
			min = args[i];
	return min; 
}

//이렇게 하면 그나마 좋아진다.
static int min(int fristArg, int... args) { 
	int min = firstArg;
	for (int arg : args)
		if (arg < min) 
			min = arg;
	return min; 
}
```

- **가변 인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화한다.**

```java
//95퍼센트의 메서드 호출이 아래 3개일 경우, 성능을 높일 수 있다.
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```

- EnumSet의 정적 팩터리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화한다.

```java
public static <E extends Enum<E>> EnumSet<E> of(E e)
public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2)
public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3)
public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4)
public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4, E e5)
public static <E extends Enum<E>> EnumSet<E> of(E first, E... rest)
```