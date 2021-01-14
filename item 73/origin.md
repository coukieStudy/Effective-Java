#### Item 73. 추상화 수준에 맞는 예외를 던지라

**상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다.** 이를 예외 번역(exception translation)이라 한다.

```java
try {
	... // 저수준 추상화를 이용한다.
} catch (LowerLevelException e) {
	// 추상화 수준에 맞게 번역한다.
	throw new HigherLevelException(...); 
}

```

저수준 예외가 디버깅에 도움이 된다면 예외 연쇄(excep- tion chaining)를 사용하는 게 좋다.

```java
try {
	... // 저수준 추상화를 이용한다.
} catch (LowerLevelException cause) {
	// 저수준 예외를 고수준 예외에 실어 보낸다.
	throw new HigherLevelException(cause); 
}
```

##### 아래 계층의 예외를 예방하거나 스스로 처리할 수 없고, 그 예외를 상위 계층에 그대로 노출하기 곤란하다면 예외 번역을 사용하라.
##### 이때 예외 연쇄를 이용하면 상위 계층에는 맥락에 어울리는 고수준 예외를 던지면서 근본 원인도 함께 알려주어 오류를 분석하기에 좋다.
