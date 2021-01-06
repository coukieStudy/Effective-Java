# 다른 타입이 적절하다면 문자열 사용을 피하라

- 문자열로 값 타입을 대신하지 마라.
    - 수치형이면 `int`, `float`, `BigInteger`를, '예/아니오' 형이면 `boolean`을 써라.
    - 적절한 값 타입이 있다면 그걸 사용하고 없으면 만들어서 써라.
- 문자열로 열거 타입을 대신하지 마라.
- 문자열로 혼합 타입을 대신하지 마라.
    - 안 좋은 예시
    ```java
    String compoundKey = className + "#" + i.next();
    ```
    - 두 요소는 `#`로 구분된다.
    - 각각의 요소에 접근하려면 파싱해야 돼서 느리고, 귀찮고, 오류 가능성도 크다.
    - `equals`, `toString`, `compareTo` 메서드도 제공할 수 없다.
    - 이런 경우에는 private 정적 멤버 클래스를 선언해서 써라.
- 문자열로 권한을 표현하지 마라.
    - 자바 2 전에는 스레드 지역변수 기능(`ThreadLocal`)이 없어서 프로그래머들이 직접 설계해야 했다. 그 결과는 클라이언트가 제공한 문자열 키로 스레드별 지역변수를 식별하는 것으로 수렴했다.
    ```java
    public class ThreadLocal {
        private ThreadLocal() { }

        // 클라이언트가 제공한 문자열이 키가 됨
        public static void set(String key, Object value);

        public static Object get(String key);
    }
    ```
    - 만약 의도치 않게 두 클라이언트가 같은 키를 쓰게 된다면 원하는 대로 기능하지 못한다.
    - 악의적인 클라이언트가 다른 클라이언트의 값을 가져올 수 있다.
    - 이 경우에는 위조할 수 없는 키를 사용하면 된다. 이 키를 권한(capacity)이라고도 한다.
    ```java
    public class ThreadLocal {
        private ThreadLocal() { }

        // 위조 불가능한 키
        public static class Key {
            key() { }
        }

        public static Key getKey() {
            return new Key();
        }

        public static void set(Key key, Object value);
        public static Object get(Key key);
    }
    ```
    - 여기서 `set`과 `get`은 `Key`에 종속되므로 정적 메서드가 아니라 `Key`의 인스턴스 메서드로 바꿔주면 된다.
    - 그러면 `Key`는 그 자체가 스레드 지역변수가 된다.
    - 이제 `ThreadLocal`이 하는 일이 없으므로, 종래의 `Key`를 `ThreadLocal`로 대신하면 된다.
    ```java
    public final class ThreadLocal {
        public ThreadLocal();
        public void set(Object value);
        public Object get();
    }
    - `set`의 `Object`가 타입 안전하지 않으니, 매개변수화 타입으로 선언해주자.
    - 이는 문자열 기반이 아니어서 가능한 것이다.
    ```java
    public final class ThreadLocal<T> {
        public ThreadLocal();
        public void set(T value);
        public T get();
    }
    ```