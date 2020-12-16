

## Item 50. 적시에 방어적 복사본을 만들라

클라이언트가 여러분의 불변식을 깨뜨리려 혈안이 되어 있다고 가정하고 **방어적으로 프로그래밍**해야 한다.

###### 코드 50-1 기간을 표현하는 클래스 - 불변식을 지키지 못했다.

```java
public final class Period {
  private final Date start;
  private final Date end;

  /**
  * @param start 시작 시각
  * @param end 종료 시각. 시작 시각보다 뒤여야 한다.
  * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
  * @throws NullPointerException start나 end가 null이면 발생한다. 
  */
  public Period(Date start, Date end) { 
    if (start.compareTo(end) > 0)
      throw new IllegalArgumentException(
      start + " after " + end);
    this.start = start;
    this.end = end; 
  }

  public Date start() {
    return start; 
  }
  public Date end() {
    return end; 
  }
... // 나머지 코드 생략 
}
```

Date가 가변이라는 사실을 이 용하면 어렵지 않게 그 불변식을 깨뜨릴 수 있다.

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // p의 내부를 수정했다!
```

Date는 낡은 API이니 새로운 코드를 작성할 때는 더 이상 사용하면 안 된다.

자바 8 이상으로 개발해도 된다면 Instant(혹은 LocalDate Time이나 ZonedDateTime)를 사용하라.(https://d2.naver.com/helloworld/645609)

생성자에서 받은 가변 매개변수 각각을 **방어적으로 복사(defensive copy)**해야 한다. 

```java
public Period(Date start, Date end) {
  this.start = new Date(start.getTime());
  this.end = new Date(end.getTime());

  if (this.start.compareTo(this.end) > 0)
    throw new IllegalArgumentException(
    	this.start + " after " + this.end);
}
```

**매개변수의 유효성을 검사(아이템 49)하기 전에 방어적 복사본을 만들 고, 이 복사본으로 유효성을 검사한 점에 주목하자.** 순서가 부자연스러워 보이 겠지만 반드시 이렇게 작성해야 한다. 멀티스레딩 환경이라면 원본 객체의 유 효성을 검사한 후 복사본을 만드는 그 찰나의 취약한 순간에 다른 스레드가 원 본 객체를 수정할 위험이 있기 때문이다.

방어적 복사에 Date의 clone 메서드를 사용하지 않은 점에도 주목하자. Date 는 final이 아니므로 clone이 Date가 정의한 게 아닐 수 있다. 즉, clone이 악의 를 가진 하위 클래스의 인스턴스를 반환할 수도 있다. 

###### 코드 50-4 **Period** 인스턴스를 향한 두 번째 공격

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78); // p의 내부를 변경했다!
```

두 번째 공격을 막아내려면 단순히 접근자가 **가변 필드의 방어적 복사본을 반환하면 된다.**

```java
public Date start() {
  return new Date(start.getTime()); 
}
public Date end() {
  return new Date(end.getTime()); 
}
```

###### 핵심 정리

> 클래스가 클라이언트로부터 받는 혹은 클라이언트로 반환하는 구성요소가 가변이라면 그 요소는 반드시 방어적으로 복사해야 한다. 복사 비용이 너무 크거나 클라이언트가 그 요소를 잘못 수정할 일이 없음을 신뢰한다면 방어적 복사를 수행하는 대신 해당 구성요소를 수정했을 때의 책임이 클라이언트에 있음을 문서에 명시하도록 하자.
