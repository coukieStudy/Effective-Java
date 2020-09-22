# finalizer와 cleaner 사용을 피해라
## finalizer란?
- `java.lang.Object` 클래스에 정의된 `finalize()` 메서드를 말한다.
- Java 8 javadoc의 설명: https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#finalize--
- 가비지 컬렉터가 객체를 메모리에서 회수하는 시점에 호출된다.
- `java.lang.Object`의 `finalize()`는 아무 일도 하지 않는다.
```java
public class Object {
    // 실제 implementation
    protected void finalize() throws Throwable() { }
}
```
- 일반적으로 이 메서드는 시스템 자원을 사용하는 클래스가 오버라이드하여 시스템 자원을 반납하는 로직을 수행하는 데 사용된다.
```java
public abstract class ImageInputStreamImpl implements ImageInputStream {
    // 실제 implementation
    protected void finalize() throws Throwable {
    if (!isClosed) {
        try {
            close();
        } catch (IOException e) {
        }
    }
    super.finalize();
    }
}
```
## cleaner란?
- Java 9에서 finalizer는 그 위험성 때문에 deprecated되고 그 대안으로 cleaner가 새로 생겼다.
- `java.lang.ref`에 정의된 `Cleaner` 클래스를 말한다.
- Java 9 javadoc의 설명: https://docs.oracle.com/javase/9/docs/api/java/lang/ref/Cleaner.html
- Cleaner는 등록된 `AutoCloseable` 구현체의 레퍼런스가 없어지면 해당 객체의 cleaning action을 실행하는 역할을 한다.
```java
// 출처: Java 9 javadoc
public class CleaningExample implements AutoCloseable {
    // A cleaner, preferably one shared within a library
    private static final Cleaner cleaner = <cleaner>;

    static class State implements Runnable {

        State(...) {
            // initialize State needed for cleaning action
        }

        public void run() {
            // cleanup action accessing State, executed at most once
        }
    }

    private final State;
    private final Cleaner.Cleanable cleanable

    public CleaningExample() {
        this.state = new State(...);
        this.cleanable = cleaner.register(this, state);
    }

    public void close() {
        cleanable.clean();
    }
}
```
- try-with-resource 구문에서 `AutoCloseable` 구현체의 인스턴스를 선언하면 try-with-resource 구문을 빠져나올 때 `close()`가 자동으로 호출된다.
```java
// 출처: https://codechacha.com/ko/java-try-with-resources/
public static void main(String args[]) {
    try (
        FileInputStream is = new FileInputStream("file.txt");
        BufferedInputStream bis = new BufferedInputStream(is)
    ) {
        int data = -1;
        while ((data = bis.read()) != -1) {
            System.out.print((char) data);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
## finalizer와 cleaner는 C++의 destructor와 다르다
- 공통점: 둘 다 객체가 메모리에서 내려올 때 호출된다.
- 차이점
    - destructor는 언제 호출될지 알 수 있다. 왜냐하면 C++에선 객체가 언제 파괴될지가 명확하기 때문이다.
    - finalizer나 cleaner는 호출 시점을 알 수 없다. 왜나하면 Java에서 객체를 파괴하는 건 순전히 GC에 달렸고, GC가 언제 일어날지는 알 수 없기 때문이다.

## finalizer와 cleaner를 쓰면 안 되는 이유
### 언제 호출될지 모른다
- 시스템 자원은 한정되어 있다.
- 객체가 소유한 시스템 자원의 해제를 전적으로 finalizer나 cleaner한테 맡기면, 나중에 자원 부족으로 오류가 발생할 수 있다.
- GC의 동작은 GC 구현에 따라 달라지므로 테스트가 불가능하다.
### 호출이 될지도 보장할 수 없다
- 객체의 레퍼런스가 없어지기 전에 프로그램이 종료되거나, GC가 한번도 수행되지 않는다면 finalizer나 cleaner는 수행되지 못한다.
- DB 트랜잭션과 같이 상태를 영구적으로 수정하는 작업에 매우 위험할 수 있다.
- 이상한 메서드로 finalizer나 cleaner 호출을 보장하려고 하지 마라
    - `System.gc()`: GC를 수행한다. -> 수집 대상이 아니면 finalizer가 호출되지 않는다.
    - `System.runFinalization()`: 수집 대상 객체들의 finalizer를 호출한다. -> 위와 마찬가지.
    - `System.runFinalizersOnExit()`과 `Runtime.runFinalizersOnExit()`: 프로세스가 끝날 때 모든 객체를 가비지로 수집한다. -> 프로세스를 강제 종료하는데 그 때 하필 어떤 객체가 시스템 자원으로 뭔가 하고 있었다면? 예상치 못한 결과를 초래할 수 있다.
### 예외가 발생해도 무시된다
- finalizer 동작 중 잡히지 않는 예외 (uncaught exception)가 발생하면 스레드 중단 없이 그냥 넘어간다. 경고조차 출력하지 않는다. 이 과정에서 객체가 마무리가 덜 된 상태로 남을 수 있다.
- finalizer가 제대로 동작하지 못한 객체는 이후에 어떻게 동작할지 예측할 수 없다.
- cleaner는 자신의 스레드를 통제할 수 있기 때문에 상관없다 (UncaughtExceptionHandler를 사용하면 된다는 의미로 보임).
### 성능이 구리다
- finalizer와 cleaner는 try-with-resource 구문을 이용하는 경우에 비해 50배나 느리다.
- finalizer와 cleaner는 GC의 효율을 떨어뜨리기 때문에 성능을 구리게 만든다. (출처: https://stackoverflow.com/questions/2860121/why-do-finalizers-have-a-severe-performance-penalty)
    - 자바에서 생성된 객체들은 오래 살아남을수록 점점 더 old한 generation 영역으로 옮겨진다 (Eden -> Survivor 0 -> Survivor 1 -> Tenure). 이 과정을 풀어서 쓰면 다음과 같다:
        1. 새 영역으로 살아남은 객체들을 복사한다.
        2. 기존 영역의 메모리를 싹 비운다.
    - 만약 제거할 객체들 중 finalizer 호출이 필요한 객체가 없다면 과정 2는 매우 빠르게 수행될 수 있다.
    - 하지만 만약 finalizer 호출이 필요한 객체가 있다면 과정 2는 이렇게 쪼개진다:
        - 제거할 객체들을 순회하며, 만약 그 객체가 finalizer 호출이 필요하면 바로 제거하지 않고 별개의 스레드에 큐잉한다.
        - 나중에 해당 스레드가 finalizer를 호출한 후 해당 객체를 제거한다.
    - 게다가 finalizer를 호출하는 스레드는 실행 우선순위가 낮기 때문에 해당 객체들이 메모리에 더 길게 상주하게 되고, 그 결과 GC가 더 자주 수행되게 된다.
- 안전망 형태로 사용하는 것은 괜찮다. 약 5배 정도의 성능 희생이 있다.
### 보안 문제를 일으킬 수 있다 (finalizer 한정)
- 다음과 같은 클래스가 있다고 해보자.
```java
public class DatabaseConnector throws NotAuthorizedException {
    public DatabaseConnector(Authorization a) {
        // 권한이 유효하지 않으면 예외 발생
        if (!a.isValid()) {
            throw NotAuthorizedException;
        }
    }

    public QueryResult executeQuery(String query) {
        // 쿼리 실행 결과를 반환하는 로직
    }
}
```
- 만약 어떤 나쁜 사람이 위 클래스를 상속한 하위 클래스를 작성해서 코드 인젝션을 한다고 해보자.
```java
public class EvilDatabaseConnector extends DatabaseConnector {
    static EvilDatabaseConnector connector;

    public EvilDatabaseConnector(Authorization a) { 
        super(a);
    }

    public void sendMe(QueryResult queryResult) {
        // 쿼리 실행 결과를 나한테 보내는 로직
    }

    protected void finalize() {
        connector = this;
        String evilQuery = "SELECT * FROM USER"
        QueryResult queryResult = executeQuery(evilQuery);
        sendMe(queryResult);
    }

    public static void main(String[] args) {
        try {
            new EvilDatabaseConnector(null);
        } catch (Exception e) { }
        System.gc();
    }
}
```
- 이렇게 하면 `DatabaseConnector` 생성자의 입력값 검사 과정에서 예외가 발생하고, 덜 만들어진 객체는 GC 대상이 된다. 그리고 GC 시점에서 finalizer가 호출된다.
- 이 객체는 static 변수로 지정되어 평생 GC 대상이 되지 않으며, 부모 클래스의 어떤 메서드도 사용할 수 있게 된다.
- 이러한 공격을 방지하려면 빈 finalizer를 만들고 `final`로 선언하면 된다.

## 대안은 무엇인가
- 해당 클래스를 `AutoCloseable`의 구현체로 만든다.
- 해당 클래스의 객체의 사용자는 명시적으로 `close()` 메서드를 호출하도록 한다 (try-with-resource 구문을 사용해야 안전하다).
- 각 인스턴스는 자신의 `close()` 메서드가 호출되었는지를 기록하면 좋다.

## 그럼 finalizer와 cleaner는 어디에 쓰라고 있는 것인가
### 안전망
- 자원 소유자가 `close()`를 실수로 까먹고 호출하지 않는 것에 대비하기 위해 사용할 수 있다.
- `FileInputStream`과 `FileOutputStream` 같은 클래스들은 `close()`와 finalizer가 모두 구현되어 있다.
### 네이티브 피어의 회수
- 네이티브 피어: 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체.
- 네이티브 메서드: C나 C++ 같은 네이티브 프로그래밍 언어로 작성된 메서드. 자바에서는 JNI (Java Native Interface)라는 것을 통해 자바 프로그램이 네이티브 메서드를 호출할 수 있도록 지원한다 (아이템 66 참고).
- 네이티브 피어는 자바 객체가 아니므로 GC가 회수하지 못한다. 따라서 네이티브 피어와 연결된 자바 객체에 finalizer를 정의하거나 cleaner를 사용해 네이티브 피어를 회수하도록 할 수 있다.
- 성능 저하를 감당할 수 있을 때에만 사용하자.