# 자바 직렬화의 대안을 찾으라

- 자바 직렬화로 인한 보안 문제는 실제로 일어나고 있는 일이다.
- 직렬화가 왜 위험한가? `OutputInputStream`의 `readObject` 메서드 때문.
    - `readObject` 메서드를 이용하면 클래스패스 안의 거의 모든 타입의 객체를 만들 수 있다.
    - 즉 대부분의 타입들의 코드 전체가 공격 범위에 들어갈 수 있다.
- 역직렬화 과정에서 위험한 동작을 수행할 수 있는 메서드를 가젯이라고 한다.
    - 여러 가젯을 함께 사용하여 가젯 체인을 구성할 수도 있다.
    - 네이티브 코드를 맘대로 실행할 수 있는 가젯 체인이 존재할 때도 있다.
- 단순히 역직렬화에 오래 걸리는 짧은 스트림을 통해 공격할 수도 있다.
    - 예시
    ```java
    static byte[] bomb() {
        Set<Object> root = new HashSet<>();
        set<object> s1 = root;
        set<object> s2 = new hashset<>();
        for (int i = 0; i < 100; i++) {
            set<object> t1 = new hashset<>();
            set<object> t2 = new hashset<>();
            t1.add("foo"); // make it not equal to t2
            s1.add(t1);
            s1.add(t2);
            s2.add(t1);
            s2.add(t2);
            s1 = t1;
            s2 = t2;
        }
        return serialize(root);
    }
    ```
    - `HashSet`을 역직렬화하려면 `hashCode` 메서드를 호출해야 한다.
    - 위 코드에서 `root`는 깊이가 100단계까지 만들어지기 때문에 역직렬화하기 위해 `hashCode`를 2^100 번 이상 호출해야 한다.
- 대안: 자바 직렬화를 사용하지 말자. 대신 JSON이나 프로토콜 버퍼를 쓰자.
    - JSON: 텍스트 기반이라 읽기 좋다.
    - 프로토콜 버퍼: 이진 표현이라 효율이 높고 데이터 스키마를 강요할 수 있다.
- 레거시 때문에 자바 직렬화를 쓸 수밖에 없다면 신뢰할 수 없는 데이터는 절대 역직렬화하지 말자 (특히 자바 RMI)
    - 역직렬화된 데이터가 안전한지 확신할 수 없다면 객체 역직렬화 필터링을 사용하자. 특정 클래스를 받아들이거나 거부할 수 있다.