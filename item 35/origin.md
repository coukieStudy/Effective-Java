## Item 35 ordinal 메서드 대신 인스턴스 필드를 사용하라

개발자는 열거 타입 상수와 연결된 정숫값이 필요하면 ordinal 메서드를 이용하고 싶은 유혹에 빠진다.

합주단의 종류를 연주자가 1명인 솔로(solo)부터 10명인 디텍트(dectet)까지 정의한 열거 타입

```java
public enum Ensemble {
  SOLO, DUET, TRIO, QUARTET, QUINTET, SEXTET, SEPTET, OCTET, NONET, DECTET;
	public int numberOfMusicians() { return ordinal() + 1; } 
}
```

상수 선언 순서를 바꾸는 순간 numberOfMusicians가 오동작하며 8중주(octet) 상수가 이미 있으니 똑같이 8명이 연주 하는 복4중주(double quartet)는 추가할 수 없다. 값을 중간에 비워둘 수도 없다.

해결책 : 열거 타입 상수에 연결된 값은 **ordinal** 메서드로 얻지말고, 인스턴스 필드에 저장하자.

```java
public enum Ensemble {
	SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5), SEXTET(6),
  SEPTET(7), OCTET(8), DOUBLE_QUARTET(8), NONET(9), DECTET(10), 
  TRIPLE_QUARTET(12);
	
  private final int numberOfMusicians;
	Ensemble(int size) { this.numberOfMusicians = size; }
	public int numberOfMusicians() { return numberOfMusicians; }
}
```
