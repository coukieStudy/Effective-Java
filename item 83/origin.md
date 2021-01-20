# 지연 초기화는 신중히 사용하라

- 지연 초기화(lazy initialization): 필드 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법
- 주로 최적화 용도로 쓰이지만, 인스턴스 초기화 때 발생하는 순환 문제를 해결하는 효과도 있다 (https://www.baeldung.com/circular-dependencies-in-spring).
- 다른 모든 최적화와 마찬가지로, 지연 초기화 역시 왠만하면 하지 말자.
    - 초기화 비용은 줄지만 필드에 접근하는 비용은 커진다.
    - 초기화가 이뤄지는 비율, 실제 초기화에 드는 비용, 각 필드가 호출되는 빈도에 따라 지연 초기화가 실제로는 성능을 느리게 할 수도 있다.
- 멀티스레드 환경에서는 지연 초기화를 하기 까다롭다. 지연 초기화하는 필드를 반드시 동기화해야 하기 때문이다.
    - 일반적인 초기화 예시
    ```java
    private final FieldType field = computeFieldValue();
    ```
    - `synchronized` 키워드를 사용한 지연 초기화 예시
    ```java
    private FieldType field;

    private synchronized FieldType getField() {
        if (field == null)
            field = computeFieldValue();
        return field;
    }
    ```
    - 정적 필드도 마찬가지로 동기화할 수 있다.
- 성능 때문에 정적 필드를 지연 초기화해야 한다면 지연 초기화 홀더 클래스 관용구를 사용하자.
    - 예시
    ```java
    private static class FieldHolder {
        static final FieldType field = computeFieldValue();
    }

    // 클래스는 클래스가 처음 쓰일 때 초기화된다는 특성을 이용한다.
    private static FieldType getField() { return FieldHolder.field; }
    ```
    - 이 관용구는 동기화가 필요 없어 성능이 느려지지 않는다. VM이 클래스를 초기화할 때만 필드 접근을 동기화하기 때문이다.
- 성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사 관용구를 사용하자.
    - 예시
    ```java
    // 필드 초기화 이후에 동기화하지 않으므로 volatile로 선언해야 한다.
    private volatile FieldType field;

    private FieldType getField() {
        FieldType result = field;
        // 첫 번째 검사에서는 락을 사용하지 않는다.
        if (result != null) {
            return result;
        }

        // 두 번째 검사에서는 락을 사용한다.
        synchronized(this) {
            if (field == null) {
                field = computeFieldValue();
            }
            return field;
        }
    }
    ```
    - 왜 `result` 지역변수가 필요할까? 그렇게 함으로서 그 필드를 딱 한 번만 읽도록 보장한다고 한다. (참고: https://stackoverflow.com/questions/21613098/java-local-vs-instance-variable-access-speed)
    - 정적 필드의 경우에도 이중검사 관용구를 사용할 수 있지만, 그냥 홀더를 사용하는 게 낫다.
    - 반복해서 초기화해도 상관없을 때는 단일검사 관용구라는 변종을 사용할 수 있다.
    ```java
    private volatile FieldType field;

    private FieldType getField() {
        FieldType result = field;
        if (result == null)
            field = result = computeFieldValue();
        return result;
    }
    ```
- 위 초기화 기법들은 기본 타입 필드에도 적용할 수 있다.
    - 수치 기본 타입 필드에 적용한다면 필드 값 검사를 `null` 대신 `0`으로 하면 된다.
    - 모든 스레드가 필드 값을 다시 계산해도 상관없고 필드의 타입이 `long`이나 `double`이 아닌 기본 타입이라면 필드 선언에서 `volatile` 키워드를 없애도 된다. (짜릿한 단일검사 관용구)
