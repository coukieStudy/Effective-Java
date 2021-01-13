#### Item 70. 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라

1. **호출하는 쪽에서 복구하리라 여겨지는 상황이라면 검사 예외를 사용하라.**

   이는 밖에서 처리하도록 강제함. (throws IOException) 즉, API 사용자가 예외를 처리해야함

2. **프로그래밍 오류를 나타낼 때는 런타임 예외를 사용하자.**

   프로그램에서 비검사 예외나 에러를 던졌다는 것은 복구가 불가능하거나 더 실행해봐야 득보다는 실이 많다는 뜻

   예) 배열의 인덱스는 0에서 ‘배열 크기 -1’ 사이여야 한다. ArrayIndexOutOfBoundsException이 발생했다는 건 이 전제조건이 지켜지지 않았다는 뜻

   **복구 가능하다고 믿는다면 검사 예외를, 그렇지 않다면 런타임 예외를 사용하자**

3. **에러는 보통 JVM이 자원 부족, 불변식 깨짐 등 더 이상 수행을 계속할 수 없 는 상황을 나타낼 때 사용한다.**

   구현하는 비검사 throwable은 모두 RuntimeException의 하위 클래스여야 한다. Error는 업계 규약 같아서 건들지 않는게 좋다.

   
![img](https://howtodoinjava.com/wp-content/uploads/2013/04/exceptionhierarchy3-8391226.png)

   

   Exception, RuntimeException, Error를 상속하지 않는 throwable을 만들 수도 있다. 이는 사용하지 말자! 이로울게 없다.
