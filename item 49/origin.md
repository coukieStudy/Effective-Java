# Item 49. 매개변수가 유효한지 검사하라.



**매개변수 검사를 하지 않는다면?** 메서드가 수행되는 중간에 모호한 예외 발생 or 메서드는 수행되나 잘못된 결과 반환.



## 생성자 몸체가 실행 전에 매개변수 유효성을 검사해야 한다.

### 제약과 예외 문서화

**public, protected 메서드는 @throws 자바독 태그를 사용해 예외를 문서화하자.**

```java
 * <p>All methods and constructors in this class throw
 * {@code NullPointerException} when passed
 * a null object reference for any input parameter.      
 public class BigInteger extends Number implements Comparable<BigInteger> {
    /**
     * Returns a BigInteger whose value is {@code (this mod m}).  This method
     * differs from {@code remainder} in that it always returns a
     * <i>non-negative</i> BigInteger.
     *
     * @param  m the modulus.
     * @return {@code this mod m}
     * @throws ArithmeticException {@code m} &le; 0
     * @see    #remainder
     */
    public BigInteger mod(BigInteger m) {
        if (m.signum <= 0)
            throw new ArithmeticException("BigInteger: modulus not positive");

        BigInteger result = this.remainder(m);
        return (result.signum >= 0 ? result : result.add(m));
    }
}
```

m이 null일 경우 NullPointerException 발생을 설명하지 않은 이유? **클래스 수준 주석은 그 클래스의 모든 public 메서드에 적용되므로 각 메서드에 일일이 기술하는 것보다 깔끔한 방법이다.** 



### java method를 사용한 검사

java 7의 **java.util.Objects.requireNonNull** : null 검사 + 예외 메세지 지정 가능 + 입력값 그대로 반환

```java
this.strategy = Objects.requireNonNull(strategy, "NPE: strategy")
```



java 9의 **checkFromIndexSize, checkFromToIndex, checkIndex** : 범위 검사 기능



### public이 아닌 메서드라면 assert(단언문)을 사용한 검사

[자바에서 Assert 사용하기](!https://offbyone.tistory.com/294)

[Assertions In Java – Java Assert Tutorial With Code Examples](!https://www.softwaretestinghelp.com/assertions-in-java/)

```java
// 재귀 정렬용 private 도우미 함수
private static void sort(long a[], int offset, int length){
    assert a != null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;
    ... // 계산 수행
}
```

- 실패하면 AssertionError throw
- 런타임에 아무런 성능 저하도 없다. (단 런타임시에 주는 플래그에 따라 성능에 영향을 미칠 수도 있다.)



### 메서드가 사용하지 않고 나중에 쓰기 위해 저장하는 경우에도 검사해야한다.

```java
static List<Integer> intArrayAsList(int[] a){
    Objects.requireNonNull(a);    
    return new AbstractList<>(){...}; // a 사용해서..
}
```

나중에 사용하려할 때 에러가 발생하면 디버깅이 어려워진다.

특히나 생성자의 매개변수의 유효성 검사는 클래스 불변식을 어기는 객체가 만들어지지 않게 하는 데 꼭 필요하다.

```java
public BigInteger(byte[] val, int off, int len) { // BigInteger 생성자 중 하나 예시
  if (val.length == 0) {
    throw new NumberFormatException("Zero length BigInteger");
  }
  Objects.checkFromIndexSize(off, len, val.length);

  if (val[off] < 0) {
    mag = makePositive(val, off, len);
    signum = -1;
  } else {
    mag = stripLeadingZeroBytes(val, off, len);
    signum = (mag.length == 0 ? 0 : 1);
  }
  if (mag.length >= MAX_MAG_LENGTH) {
    checkRange();
  }
}
```



### 예외

1. 유효성 검사 비용이 지나치게 높거나 실용적이지 않은 경우
2. 계산 과정에서 암묵적으로 검사가 수행될 경우
   - `Collections.sort(List)` 리스트 안의 객체들은 모두 상호 비교될 수 있어야한다. 하지만 실행 전이 아니라 정렬 과정에서 이 비교가 이루어진다.



