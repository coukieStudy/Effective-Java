#### Item 71. 필요 없는 검사 예외 사용은 피하라

검사 예외는 발생한 문제를 프로그래머가 처리하여 안전성을 높이게끔 해준다. 물론, 검사 예외를 과하게 사용하면 오히려 쓰기 불편한 API가 된다.

검사 예외를 사용하는 경우

1. API를 제대로 사용할 경우에도 예외가 발생할 경우
2. 프로그래머가 의미 있는 조치를 취할 수 있는 경우 (복구 가능한 예외)

위 두가지 상황이 아닐 경우는 비검사 예외를 사용하자.

검사 예외를 회피하는 방법으론 Optinal값 제공, 상태검사,의존 메서드 제공등이 있다.

상태검사 refactoring의 예
```java
try {
  obj.action(args);
} catch (TheCheckedException e) {
	... // 예외 상황에 대처한다. 
}
```

```java
if (obj.actionPermitted(args)) {
  obj.action(args);
} else {
	... // 예외 상황에 대처한다. 
}
```

