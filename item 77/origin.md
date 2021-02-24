# Chapter 77 예외를 무시하지 말라

- 잘못된 예시

```java
try{
		..
} catch (SomeException e) {
	//아무 일도 안함
}
```

- 만약 예외를 무시하기로 했다면, Catch 블록 안에 그렇게 결정한 이유를 주석으로 남기고 예외 변수의 이름도 ignored로 바꿔놓도록 하자.

```java
int numColors = 4; // 기본값. 어떤 지도라도 이 값이면 충분하다. 
try {
	numColors = f.get(1L, TimeUnit.SECONDS);
} catch (TimeoutException | ExecutionException ignored) {
	// 기본값을 사용한다(색상 수를 최소화하면 좋지만, 필수는 아니다). 
}
```

- **이런 식으로 하게 되면, 문제의 원인과 아무 상관없는 곳에서 갑자기 죽어버릴 수 있다.**