# Chapter 29 이왕이면 제너릭 타입으로 만들라.

## 제너릭 타입으로 만들어야 하는 이유

- 제너릭 타입으로 만들면 컴파일 타임에 오류를 잡을 수 있다. 만약 제너릭 타입으로 선언하지 않게 된다면 런타임에 오류가 나게 되어 원하는 동작을 하지 않게 된다.
- Code
    - [https://github.com/HaeUlNam/effective-java-3e-source-code/blob/master/src/effectivejava/chapter5/item29/StackUsingObject.java](https://github.com/HaeUlNam/effective-java-3e-source-code/blob/master/src/effectivejava/chapter5/item29/StackUsingObject.java)
    - [https://github.com/HaeUlNam/effective-java-3e-source-code/blob/master/src/effectivejava/chapter5/item29/client.java](https://github.com/HaeUlNam/effective-java-3e-source-code/blob/master/src/effectivejava/chapter5/item29/client.java)

![Untitled](https://user-images.githubusercontent.com/26040955/100615420-ec550480-335a-11eb-80d2-24a383ba21fa.png)

## 제너릭 E로는 배열을 만들 수 없다.

```java
elements = new E[DEFAULT_INITIAL_CAPACITY];
```

## 실체화 불가 타입인 E로 배열을 만들 수 있는 방법

1. 제너릭 배열 **생성** 시에 E배열로 변환
    - 아래와 같이, Object 배열을 생성한 다음 제너릭 배열로 형변환하면 컴파일 시에 오류 대신 경고를 내보내는데, 일반적으로는 타입 안전하지 않다.

    ![Untitled 1](https://user-images.githubusercontent.com/26040955/100615471-ff67d480-335a-11eb-8a89-01160a87119d.png)

    - 문제의 배열 elements는 private 필드에 저장되고, 클라이언트로 반환되거나 다른 메서드에 전달되는 일이 전혀 없다. **push, pop으로는 모두 E만 전달되기에 안전하다.**
    - 따라서 아래 SuppressWarnings로 숨길 수 있다.

    ```java
    @SuppressWarnings("unchecked")
    ```

2. elements 필드 타입을 아래와 같이 Object[]로 변경

```java
private E[] elements; //AS-IS
private Object[] elements; //TO-BE
```

- push와 pop할 때 E로 변환해준다. → 이것또한 push는 항상 E만 해주기 때문에, 안전하다.

```java
public E pop() {
    if (size == 0)
        throw new EmptyStackException();

    // push requires elements to be of type E, so cast is correct
    @SuppressWarnings("unchecked") E result =
            (E) elements[--size];

    elements[size] = null; // Eliminate obsolete reference
    return result;
}
```

### 첫번째(생성 시 E[]로 변환) vs 두번째(Object[]로 선언하고 pop할 때 E로 변환해서 내보냄)

- 첫번째 장점
    - 가독성이 더 좋다.
    - 코드도 더 짧다. 첫번째 방식은 형변환을 배열 생성 시 한번만 하면 되지만, 두 번째 방식에서는 배열에서 원소를 읽을 때마다 변환해줘야 한다.
    - 따라서, 현업에서는 첫 번째 방식을 더 선호하며 자주 사용

- 첫번째 단점
    - E가 Object가 아닌 이상 배열의 런타임 타입이 컴파일타입 타입과 달라 [힙 오염(Heap pollution)](https://www.geeksforgeeks.org/what-is-heap-pollution-in-java-and-how-to-resolve-it/)을 일으킨다.
    - [https://junhyunny.blogspot.com/2019/02/heap-pollution.html](https://junhyunny.blogspot.com/2019/02/heap-pollution.html)

## Bounded Type Generic

```java
class DelayQueue<E extends Delayed> implements BlockingQueue<E>
```

- 위처럼 선언하면 Delayed의 하위 타입만 받는 다는 뜻이다. 이렇게 하면, Client는 내부에 원소가 무엇인지 신경쓰지 않고 Delayed의 함수를 모두 사용 가능.
