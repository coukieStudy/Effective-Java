# 커스텀 직렬화 형태를 고려해보라.

### 기본 직렬화 형태에 적합한 후보

객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태를 선택해도 무방하다. 예를 들어 사람의 이름을 표현하는 클래스는 괜찮을 것이다.

```java
public class Name implements Serializable {
    /**
     * 성. null이 아니어야 한다.
     * @serial
     */
    private final Stirng lastName;

    /**
     * 이름. null이 아니어야 한다.
     * @serial
     */
    private final String firstName;

    /**
     * 중간이름. 중간이름이 없다면 null
     * @serial
     */
    private final String middleName;

    ... // 나머지 코드는 생략
}
```

기본 직렬화 형태가 적합해도 불변식 보장과 보안을 위해 `readObject` 메서드를 제공해야 하는 경우가 많다. 앞에서 살펴본 Name 클래스를 예로 들면, lastName과 firstName 필드가 null이 아님을 `readObject` 메서드가 보장해야 한다.



### 기본 직렬화 형태에 적합하지 않은 클래스

```java
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;

    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
    // ... 생략
}
```

위 클래스는 논리적으로 문자열을 표현했고 물리적으로는 문자열들을 이중 연결 리스트로 표현했다. 

이 경우에는 기본 직렬화 형태를 사용하면 크게 네 가지 문제가 생긴다.

- 공개 API가 현재의 내부 표현 방식에 영구히 묶인다.
  - 예를 들어, 향후 버전에서는 연결 리스트를 사용하지 않게 바꾸더라도 연결리스트로 표현된 입력도 처리할 수 있어야 한다.
- 너무 많은 공간을 차지할 수 있다. 
  - 위의 예에서 직렬화 형태는 직렬화할 가치가 없는 모든 엔트리와 연결 정보까지 모두 기록한다. 저장, 전송 속도 저하.
- 시간이 많이 걸린다.
  - 직렬화 로직은 객체 그래프의 위상에 관한 정보를 알 수 없으니, 직접 순회할 수밖에 없다.
- 스택 오버플로를 발생시킨다.
  - 기본 직렬화 형태는 객체 그래프를 재귀 순회한다. 호출 정도가 많아지면 이를 위한 스택이 감당하지 못할 것이다.



### 합리적 직렬화 형태

물리적 상세표현을 제외한 논리적 구성만을 직렬화에 담는 것!

```java
public final class StringList implements Serializable {
    private transient int size = 0;
    private transient Entry head = null;

    // 이번에는 직렬화 하지 않는다.
    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }

    // 문자열을 리스트에 추가한다.
    public final void add(String s) { ... }

    /**
     * StringList 인스턴스를 직렬화한다.
     */
    private void writeObject(ObjectOutputStream stream)
            throws IOException {
        stream.defaultWriteObject();
        stream.writeInt(size);

        // 모든 원소를 순서대로 기록한다.
        for (Entry e = head; e != null; e = e.next) {
            stream.writeObject(e.data);
        }
    }

    private void readObject(ObjectInputStream stream)
            throws IOException, ClassNotFoundException {
        stream.defaultReadObject();
        int numElements = stream.readInt();

        for (int i = 0; i < numElements; i++) {
            add((String) stream.readObject());
        }
    }
    // ... 생략
}
```

`transient` 키워드가 붙은 필드는 기본 직렬화 형태에 포함되지 않는다. 클래스의 모든 필드가 `transient`로 선언되었더라도 `writeObject`와 `readObject` 메서드는 각각 `defaultWriteObject`와 `defaultReadObject` 메서드를 호출해야 한다. 이렇게 해야 향후 릴리즈에서 `transient`가 아닌 인스턴스 필드가 추가되더라도 상위와 하위 모두 호환이 가능하기 때문이다.

- 해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 transient 한정자를 생략한다. 그래서 커스텀 직렬화를 사용한다면 대부분의 필드를 transient로 선언해야 한다.
- 기본 직렬화를 사용한다면 transient 필드들은 역직렬화될 때 기본값으로 초기화된다. 기본값을 사용하기를 원하지 않으면 readObject 메서드에서 defaultReadObject 메서드를 호출한 다음 원하는 값으로 지정하거나 그 값을 처음 사용할 때 초기화하면 된다.



### 직렬화의 동기화

기본 직렬화 사용 여부와 상관없이 직렬화에도 동기화 규칙을 적용해야 한다. 예를 들어 모든 메서드를 `synchronized`로 선언하여 스레드 안전하게 만든 객체에 기본 직렬화를 사용한다면, `writeObject`도 아래처럼 수정해야 한다.

```java
private synchronized void writeObject(ObjectOutputStream stream)
        throws IOException {
    stream.defaultWriteObject();
}
```



### SerialVersionUID

어떤 직렬화 형태를 선택하더라도 직렬화가 가능한 클래스에는 SerialVersionUID(이하 SUID)를 명시적으로 선언해야 한다. 물론 선언하지 않으면 자동 생성되지만 런타임에 이 값을 생성하느라 복잡한 연산을 수행해야 한다.

```java
private static final long serialVersionUID = // 무작위로 고른 long 값
```

- serialver utility
- 꼭 고유할 필요는 없다.
- 기존버전과의 호환성을 유지하고 싶으면 똑같이.
- 구버전과 직렬화된 인스턴스들과의 호환성을 끊으려는 경우를 제외하고는 SerialVersionUID를 절대 수정하지 말자!
