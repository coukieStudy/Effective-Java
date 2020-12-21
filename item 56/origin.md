# 공개된 API요소에는 항상 문서화 주석을 작성하라

자바에는 Javadoc이라는 유틸리티가 있어서 소스코드 파일에서 문서화 주석이라는 특수한 형태로 기술된 설명을 추려서 AP 문서로 변환해준다.



### 문서화 주석을 달아야 하는 범위

* 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언
  * 기본 생성자에는 주석을 달 수 없으므로, 공개된 API에는 기본 생성자를 막아야 한다.
* 직렬화할 수 있는 클래스라면 직렬화 형태에 관해서도 적어야 한다.
* 유지 보수를 위해서 공개되지 않은 클래스, 인터페이스, 생성자, 메서드, 필드에도 문서화 주석을 달면 좋다.



### 문서화 주석의 내용


* 메서드의 주석은 해당 메서드와 클라이언트 사이의 규약을 기술해야 한다.
* 상속용 클래스의 메서드가 아니라면, 그 메서드의 동작 방식(how)이 아니라 무엇을 하는지(what)을 기술해야 한다.
* 상속용으로 클래스를 설계할 때에는 자기사용 패턴에 대해서도 문서를 남겨서 해당 메서드를 올바르게 재정의하는 방법을 명시해야 한다.
* 해당 메서드를 호출하기위한 전제조건(precondition)과 사후조건(postcondition), 그리고 부작용을 명시해야 한다.
  * 전제조건의 예시: 해당 메서드가 던지는(throw) 비검사 예외
  * 부작용의 예시: 해당 메서드가 백그라운드 스레드를 시작시키는 경우
* 모든 매개변수에 @param 태그를 달고, 반환 타입에 @return을 달고, 모든 예외에 @throws 태그를 달면 좋다.
예시)
```java
/**
     * Returns the element at the specified position in this list.
     *
     * <p>This method is <i>not</i> guaranteed to run in constant
     * time. In some implementations it may run in time proportional
     * to the element position.
     *
     * @param  index index of element to return; must be
     *         non-negative and less than the size of this list
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index >= this.size()})
     */
E get(int index) {
  return null;
}

```
* 제네릭 타입이나 제네릭 메서드를 문서화할 때는 모든 타입 매개변수에 주석을 달아야 한다.
```java
/**
* An object that maps keys to values. A map cannot contain
* duplicate keys; each key can map to at most one value. *
* (Remainder omitted) *
* @param <K> the type of keys maintained by this map
* @param <V> the type of mapped values
*/
public interface Map<K, V> { ... }
```
* 열거 타입을 문서화할 때에는 상수들에도 주석을 달아야 한다.
```java
/**
* An instrument section of a symphony orchestra.
*/
public enum OrchestraSection {
    /** Woodwinds, such as flute, clarinet, and oboe. */
    WOODWIND,

    /** Brass instruments, such as french horn and trumpet. */
    BRASS,

    /** Percussion instruments, such as timpani and cymbals. */
    PERCUSSION,

     /** Stringed instruments, such as violin and cello. */
     STRING;
}
```
* 애너테이션 타입을 문서화할 때에는 애너테이션 타입 자체 뿐만 아니라 멤버들에도 모두 주석을 달아야 한다.
```java
/**
* Indicates that the annotated method is a test method that
* must throw the designated exception to pass.
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    /**
     * The exception that the annotated test method must throw
     * in order to pass. (The test is permitted to throw any
     * subtype of the type described by this class object.)
     */
    Class<? extends Throwable> value();
}
```
* 패키지를 설명하는 문서화 주석은 package-info.java 파일에 작성한다. 모듈 시스템을 사용한다면 모듈 관련 설명은 module-info.java 파일에 작성한다.
* 스레드 안전성과 직렬화 가능성은 반드시 API 설명에 포함되어야 한다.
* 자바독은 메서드 주석을 상속시킬 수 있다. 문서화 주석이 없는 API 요소가 있다면, 자바독이 가장 가까운 문서화 주석을 찾는다.
* 각 문서의 첫 번째 문장은 해당 요소의 요약 설명(summary description)으로 간주되며, 대상의 기능을 고유하게 기술해야 한다. 요약 설명이 일치하는 생성자, 혹은 멤버가 둘 이상 있으면 안되므로 다중 정의된 메서드들의 요약  설명을 기술할 때에 조심해야 한다.
* 요약 설명은 마침표를 기준으로 인식되므로 Mr. / Mrs. 처럼 특수한 경우에 붙는 마침표를 주의해야 한다.
* 메서드/생성자의 요약설명은 해당 메서드와 생성자의 동작을 설명하는 동사구이며 클래스/인터페이스/필드의 요약설명은 대상을 설명하는 명사절이어야 한다.


### 문서화 주석의 태그

* 자바독 유틸리티는 문서화 주석을 HTML로 번역하므로, HTML태그를 쓸 수 있다.
* {@code} 태그를 사용하면 감싼 내용을 코드용 폰트로 바꾸고, 태그 안의 다른 HTML요소나 다른 자바독 태그를 무시한다. 그래서 해당 태그 안에서는 메타 기호인 \<를 바로 사용할 수 있다.
* 여러 줄에 걸친 코드 태그를 사용하고 싶으면 다음의 형태를 사용하면 된다.
```java
<pre>{@code ... 코드 ... }</pre> 
```
* @implSpec 주석은 자기사용 패턴을 문서화하며, 해당 메서드와 하위 클래스 사이의 계약에 대한 설명이 포함되어 있어야 한다.
```java
    // Use of @implSpec to describe self-use patterns & other visible implementation details. (Page 256)
    /**
     * Returns true if this collection is empty.
     *
     * @implSpec This implementation returns {@code this.size() == 0}.
     *
     * @return true if this collection is empty
     */
public boolean isEmpty() {
    return false;
}
```
* 자바독 명령줄에서 다음 스위치를 켜야만 @implSpec 태그의 내용이 나타난다.
```java
-tag "implSpec:a:Implementation Require ments:"
```
* {@literal} 태그를 이용하면 HTML 메타문자인 \<, \>, \& 등을 사용할 수 있다.
* Java 9 부터는 자보독이 생성한 HTML 문서에 검색 기능이 추가되었다. 클래스, 메서드, 필드 같은 요소의 색인은 자동으로 만들어지며, 기타 중요한 용어는 {@index} 태그를 사용해 만들 수 있다.
* {@inheritDoc} 태그를 사용해서 상위 타입의 문서화 주석 일부를 상속할 수 있다. 유지보수의 부담은 줄일 수 있지만, 사용하기 까다롭고 제약 사항도 있다.


