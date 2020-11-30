# 멤버 클래스는 되도록 static으로 만들라
## Nested class의 종류
- Static member class
- Inner class
    - Non-static member class
    - Anonymous class
    - Local class
## Static member class
- 일반 클래스와 똑같다. 두 가지만 빼고.
    - Outer class 안에 선언된다.
    - Outer class의 private 멤버에 접근할 수 있다.
- Outer class와 함께 쓰일 때 유용한 헬퍼 클래스로 많이 쓰인다.
    - ex) Calculator (계산기) 클래스의 static member class로써 Operation enum 클래스 (PLUS, MINUS, ...)
## (Non-static) member class
- Outer class 인스턴스와 암묵적으로 연결된다.
    - `클래스명.this` 용법을 통해 outer class의 멤버를 참조할 수 있다. (정규화된 this)
- 따라서 의미상 nested class의 인스턴스가 outer class 인스턴스와 독립적으로 존재할 수 있다면 static member class로 만드는 게 좋다.
- Member class 인스턴스와 바깥 인스턴스 간의 관계는 member class 인스턴스 생성 때 확립되며 변경할 수 없다. 그 관계 정보는 member class 안에 만들어진다. (member class의 final 필드로서 존재함)
    - 그러므로 member class는 static member class에 비해 메모리도 더 먹고 생성 시간도 더 걸린다.
- Member class는 어댑터를 정의할 때 자주 쓰인다.
    - 어댑터 패턴: https://yaboong.github.io/design-pattern/2018/10/15/adapter-pattern/
    - ex) Map 인터페이스 구현체들이 `keySet`, `entrySet`, `values`로 반환하는 컬렉션 뷰, `Set`과 `List`의 iterator
- Member class에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static member class로 만들어라. 그렇지 않으면
    - 시간과 공간이 낭비된다.
    - GC가 바깥 인스턴스를 수거하지 못해 메모리 누수가 생길 수 있다.
- 예시: `Map.Entry`
    - 엔트리는 맵과 연관되어 있지만 맵에 직접 접근할 일은 없다.
    ```java
    Interface Entry<K, V> {
        K getKey();
        V getValue();
        V setValue(V value);
    }
    ```
    - 그러므로 엔트리를 non-static member class로 구현할 필요가 없다. 실제로도 static으로 구현되어 있다.
    ```java
    // TreeMap 코드의 일부
    static final class Entry<K,V> implements Map.Entry<K,V> {
        K key;
        V value;
        Entry<K,V> left;
        Entry<K,V> right;
        Entry<K,V> parent;
        boolean color = BLACK;

        /**
         * Make a new cell with given key, value, and parent, and with
         * {@code null} child links, and BLACK color.
         */
        Entry(K key, V value, Entry<K,V> parent) {
            this.key = key;
            this.value = value;
            this.parent = parent;
        }
        ...
    }
    ```
- member class가 public이나 protected면 더욱 중요하다. 나중에 고칠 일이 없게 미리 static을 붙여주자.
## Anonymous class
- 특정 클래스 혹은 인터페이스를 상속/구현하며 임시적으로 만들어지는 클래스.
    - 람다 이전에 함수형 프로그래밍을 흉내내기 위해 사용되었다.
    ```java
    // anonymous class
    Runnable runnable = new Runnable() {
        @Override
        public void run() {...}
    }

    // lambda
    Runnable runnable = () -> {...}
    ```
    - 정적 팩토리 메서드를 구현할 때 사용할 수 있다. (ex. 코드 20-1)
- 쓰이는 시점에 선언과 동시에 인스턴스화된다.
- 코드 어디서든 만들 수 있으며 쓰이는 시점에 따라 사용할 수 있는 멤버가 달라진다.
    - 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있다.
    - 정적 문맥에서 사용할 때도 final 기본 타입 혹은 문자열이 아닌 정적 필드는 가질 수 없다.
- 사용에 제약이 많다.
    - 선언한 지점에서만 인스턴스가 만들어진다.
    - `instanceof`나 클래스 이름이 필요한 작업은 수행할 수 없다.
    - 익명 클래스를 사용하는 클라이언트는 익명 클래스가 상속한 멤버 외에는 호출할 수 없다.
    - 길면 가독성이 떨어진다.
## Local class
- 지역변수를 선언할 수 있는 곳에 선언할 수 있는 클래스. (ex. 메서드 안)
- 유효 범위도 지역변수와 같다. (ex. 메서드 안에 선언했으면 그 메서드 밖에선 사용할 수 없다)
- 다른 nested class와의 공통점
    - member class처럼 이름이 있고 반복해서 사용할 수 있다.
    - anonymous class처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있으며 정적 멤버는 final 기본 타입 혹은 문자열만 가질 수 있다.
