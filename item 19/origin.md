# Item 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라.

## 상속을 고려한 설계와 문서화

**상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야 한다.**

- 클래스의 API로 공개된 메서드에서 클래스 자신의 또 다른 메서드를 호출하고 그 호출되는 메서드가 재정의 가능 메서드일 경우
- 호출 순서, 각각의 호출 결과의 영향 등 모든 상황을 문서로 남겨야 한다.


```java
// package.java.util
// AbstractCollection

 /**
     * {@inheritDoc}
     *
     * <p>This implementation iterates over the collection looking for the
     * specified element.  If it finds the element, it removes the element
     * from the collection using the iterator's remove method.
     *
     * <p>Note that this implementation throws an
     * <tt>UnsupportedOperationException</tt> if the iterator returned by this
     * collection's iterator method does not implement the <tt>remove</tt>
     * method and this collection contains the specified object.
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     */
    public boolean remove(Object o) { ... }
```
이 경우에는 iterator 메서드를 재정의하면 remove 메서드의 동작에 영향을 줌을 확실히 알 수 있다. 상속은 이처럼 공개하지 않아될 내부 구현을 설명하게 만든다.

**@implSpec**

https://jaehun2841.github.io/2019/02/24/effective-java-item56/#%EB%AC%B8%EC%84%9C%ED%99%94-%ED%83%9C%EA%B7%B8

**hook / protected method**

- 내부 매터니즘을 문서로 남기는 것만이 유일한 방법은 아니다.
- 내부 동작 과정 중간에 끼어들 수 있는 `hook`을 잘 선별해 protected 메서드(또는 드물게 필드) 형태로 공개할 수도 있다.

```java
// package.java.util
// AbstractList

/**
     * Removes all of the elements from this list (optional operation).
     * The list will be empty after this call returns.
     *
     * <p>This implementation calls {@code removeRange(0, size())}.
     *
     * <p>Note that this implementation throws an
     * {@code UnsupportedOperationException} unless {@code remove(int
     * index)} or {@code removeRange(int fromIndex, int toIndex)} is
     * overridden.
     *
     * @throws UnsupportedOperationException if the {@code clear} operation
     *         is not supported by this list
     */
    public void clear() {
        removeRange(0, size());
    }

/**
     * Removes from this list all of the elements whose index is between
     * {@code fromIndex}, inclusive, and {@code toIndex}, exclusive.
     * Shifts any succeeding elements to the left (reduces their index).
     * This call shortens the list by {@code (toIndex - fromIndex)} elements.
     * (If {@code toIndex==fromIndex}, this operation has no effect.)
     *
     * <p>This method is called by the {@code clear} operation on this list
     * and its subLists.  Overriding this method to take advantage of
     * the internals of the list implementation can <i>substantially</i>
     * improve the performance of the {@code clear} operation on this list
     * and its subLists.
     *
     * <p>This implementation gets a list iterator positioned before
     * {@code fromIndex}, and repeatedly calls {@code ListIterator.next}
     * followed by {@code ListIterator.remove} until the entire range has
     * been removed.  <b>Note: if {@code ListIterator.remove} requires linear
     * time, this implementation requires quadratic time.</b>
     *
     * @param fromIndex index of first element to be removed
     * @param toIndex index after last element to be removed
     */
    protected void removeRange(int fromIndex, int toIndex) { ... }
    }
```

이 경우에 removeRange는 하위 클래스에서 clear의 성능에 영향을 미치는 hook이다. 따라서 protected로 공개한다.

**상속용 클래스르 시험하는 방법은 직접 하위 클래스를 만들어 보는 것이 유일하다.**

## 상속을 허용하는 클래스의 제약

- 상속용 클래스의 생성자는 직접적/ 간접적으로든 재정의 가능 메서드를 호출해서는 안된다. 

아래의 예시에서 결과는 instant를 두 번 출력하기를 기대했지만 첫 번째는 null을 출력한다. 상위 클래스의 생성자는 하위 클래스의 생성자가 인스터느 필드를 초기화하기도 전에 overrideMe를 호출하기 때문이다.

```java
// Super.java
public class Super {

    public Super() {
        // 잘못된 예
        overrideMe();
    }
    
    public void overrideMe() { }
}
```

```java
// Sub.java
public final class Sub extends Super {

    private final Instant instant;

    public Sub() {
        super();
        instant = Instant.now();
    }

    @Override
    public void overrideMe() {
        System.out.println(instant);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```

- Clonable, Serializable 인터페이스를 구현한 클래스는 상속할 수 있게 설계하는 것은 좋지 않다! 왜냐하면 clone, readObject 메서드는 생성자와 비슷한 효과를 내기(새로운 객체를 만든다) 때문이다.
- 일반적인 구체 클래스일 때도 클래스이 변화가 생길 때마다 하위 클래스의 오동작 유발할 수 있다.

> 상속용으로 설계하지 않은 클래스는 상속을 금지하자!


## 상속을 금지하는 방법

1. 클래스를 final로 선언하는 방법
2. 모든 생성자를 private or package-private로 선언하고 public 정적 팩터리를 만들어주는 방법이다. (item 17)
3. 상속을 꼭 써야한다면 재정의 가능 메서드를 사용하지 않게 문서화하고 재정의 가능 메서드를 호출하는 자기 사용 코드를 완전히 제거하기.
	- 클래스의 동작을 유지하면서 제거하는 방법: private 도우미 메서드로 재정의 가능 메서드의 본문을 옮기고 재정의 가능 메서드 대신 도우미 메서드를 호출하도록 한다.
