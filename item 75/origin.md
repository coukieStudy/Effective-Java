# Chapter 75 예외의 상세 메서지에 실패 관련 정보를 담으라

- 사후 분석을 위해 실패 순간의 상황을 정확히 포착해 예외의 상세 메시지에 담아야 한다.
- 실패 순간을 포착하려면 발생한 예외에 관여된 모든 매개변수와 필드의 값을 실패 메시지에 담아야 한다.

ex) IndexOutOfBoundsException: 범위의 최솟값, 최댓값, 그리고 그 범위를 벗어났다는 인덱스의 값을 담아야 한다 → 사후에 어떤 부분을 수정해야 할지 분석 가능

- IndexOutOfBoundsException 생성자의 좋은 예시 → 현재 Java 8은 이렇게 구현되어있지 않다

```java
/**
* IndexOutOfBoundsException을 생성한다. *
* @param lowerBound 인덱스의 최솟값
* @param upperBound 인덱스의 최댓값 + 1 * @param index 인덱스의 실젯값
*/
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
	// 실패를 포착하는 상세 메시지를 생성한다. 
	super(String.format("최솟값: %d, 최댓값: %d, 인덱스: %d", lowerBound, upperBound, index));
	// 프로그램에서 이용할 수 있도록 실패 정보를 저장해둔다. this.lowerBound = lowerBound; this.upperBound = upperBound;
	this.index = index;
}
```

→ 하지만 예외 메세지를 작성할 때, 적절한 정보(위와 같은..)를 넣어주는 것 중요.