# clone() 재정의는 주의해서 진행하라.

Object.java에 명세된 clone()의 일반 규약: 다음 명제들이 일반적으로 참인 의도를 가진다.

```java
1. x.clone() != x
2. x.clone().getClass() == x.getClass()
3. x.clone().equals(x)
```

### Clonable 인터페이스를 이용한 Object.clone()의 여러 문제점.

1. 일반적이지 않은 Interface 사용 방식과 객체 생성 방식
    1. 어떤 클래스에서 Object.clone() 을 호출할 때, 해당 클래스가 Clonable을 구현했으면 Object.clone()이 동작하지만, 그렇지 않을 경우 CloneNotSupportedException 을 발생시킨다. - Interface 의 이례적인 사용 방법
    2. clone()은 원래의 객체와 다른 객체를 반환하므로 생성자의 역할을 하지만, 명시적으로 생성자를 사용하지 않아서 위험하다.
    3. Clonable 인터페이스를 구현했다면 CloneNotSupportedException 이 절대 발생하지 않지만, CloneNotSupportedException 이 checked exception이기 때문에 반드시 예외처리를 해주어야 한다.
    
2. 상속과 관련한 문제가 발생할 수 있다.
    - Clonable 인터페이스를 구현한 클래스를 상속할 경우, 하위 클래스에서 super.clone()을 호출하면 상위 클래스의 객체가 반환된다. 따라서 하위 클래스에서도 clone()을 재정의하지 않으면 하위 클래스에서 clone()이 제대로 동작하지 않는다.
3. 클래스의 field 중에 가변 객체가 존재하는 경우, clone()이 의도대로 동작하지 않는다.
    1. 다음과 같은 클래스에서 주석으로 처리된 부분을 제외하고 super.clone()을 부르면 int size는 제대로 복사가 되지만, elements는 복사가 제대로 이루어지지 않는다. clone된 객체와 원본 객체가 같은 배열을 참조하게 된다.
    ```java
    public class Stack implements Cloneable {
        private Object[] elements;
        private int size = 0;
        private static final int DEFAULT_INITIAL_CAPACITY = 16;
        /*
        @Override public Stack clone() {
            try {
                Stack result = (Stack) super.clone();
                result.elements = elements.clone();
                return result;
            } catch (CloneNotSupportedException e) {
                throw new AssertionError();
            }
         }
         */
    }
    ```
    2. 배열 내의 원소가 가변 객체일 경우에는 배열의 단순 복사로 해결할 수 없다. buckets는 배열이지만, Entry는 linked list의 형태를 띄고 있다. 이 경우 buckets 뿐만 아니라 각 원소인 Entry의 원소들까지 복사를 해주어야 한다.
    ```java
    public class HashTable implements Cloneable {
        private Entry[] buckets = ...;
        private static class Entry {
            final Object key;
            Object value;
            Entry next;
            Entry(Object key, Object value, Entry next) {
                this.key = key;
                this.value = value;
                this.next = next;
            }
        }
    }
    ```
 4. final을 포기해야 할 때가 있다. 위 스택의 예시에서 elements가 final로 선언되어 있으면, clone된 객체의 elements 필드에 적절한 값을 대입할 수 없게 된다.
 5. Object.clone() 은 Thread safe하지 않다.
 
### clone()을 활용하는 방법.
 
1. 상속용 클래스에서는 Clonable을 구현하지 않는다. 이를 방지하기 위한 두 방법은 다음과 같다.
    1. Object 처럼 제대로 동작하는 clone() 을 protected로 만든 뒤, 하위클래스가 Clonable을 구현하지 않았으면 CloneNotSupportedException 를 던지도록 설계한다.
    2. clone()이 언제나 CloneNotSupportedException 을 던지게 해서 하위 클래스에서 clone()을 재정의하지 못하게 한다.
2. Cloneable을 구현하는 모든 클래스는 clone()을 재정의해야 한다.
    1. 접근 제한자는 public
    2. return type은 자기 자신
    3. super.clone()을 호출한 뒤 필요에 따른 복사 실행
3. 배열(Array)의 clone은 예외적으로 clone() 을 활용하는 것이 적절하다.
    
    
### 복사 생성자와 복사 팩터리 방식의 장점

```java
public Yum(Yum yum) { ... };
public static Yum newInstance(Yum yum) { ... };
```

1. 객체가 반드시 생성자나 정적 팩토리 아이템을 통해서만 생성된다.
2. 정상적인 final 용법과 충돌하지 않는다.
3. 불필요한 예외검사를 던지지 않는다.
4. 형변환을 별도로 하지 않아도 된다.
5. 반드시 동일한 타입의 객체가 아니라, 그 객체의 인터페이스 타입의 변수를 인수로 받을 수도 있다.
```java
//LinkedList.java
public LinkedList(Collection<? extends E> c) {
  this();
  addAll(c);
}
```
    
