#### Item 72. 표준 예외를 사용하라

자바 라이브러리는 대부분 API에서 쓰기에 충분한 수의 예외를 제공한다.

1.IllegalArgumentException -> 허용하지 않는 인수값을 던졌을때

2.IllegalStateException-> 해당 객체가 메서드를 수행할 수 없는 상태일때

3.NullPointerException-> null을 허용하지 않는 메서드에 null을 줄때

4.UnsupportedOperationException -> 호출한 메서드가 지원하지 않을때

5.IndexOutOfBoundsException -> 인덱스 범위를 넘을때

6.ConcurrentModificationException-> 동시수정이 발생했을때

**Exception**, **RuntimeException**, **Throwable**, **Error**는 직접 재사용하지 말자. 이 클래스들은 추상 클래스라고 생각하길 바란다.
