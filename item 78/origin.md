# Chapter 78 공유 중인 데이터는 동기화해 사용하라.

- Synchronized 키워드: 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장.
- 동기화
    - 배타적 실행: 한 스레드가 변경하는 중이라서 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막는 용도
    - 락 보호 하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다(동시에 접근하면 보장 못 함.)
- Java 명세상 long, double외의 변수를 읽고 쓰는 동작은 원자적이다
    - long과 double이 64bit이기 때문에, 32bit씩 끊어서 read, write
    - [https://dzone.com/articles/longdouble-are-not-atomic-in-java](https://dzone.com/articles/longdouble-are-not-atomic-in-java)
- '그럼 성능을 높이려면 원자적 데이터를 읽고 쓸 때는 동기화하지 말아야겠다'라고 생각하기 쉬운데, 위험한 발상
    - 자바 명세상 스레드가 필드를 읽을 때 항상 '수정이 완전히 반영된' 값을 얻는다고 보장하지만, 한 스레드가 저장한 값이 다른 스레드에게 '보이는가'는 보장하지 않는다.
    - 동기화는 배타적 실행 뿐만 아니라 스레드 사이의 안정적인 통신에 꼭 필요

### Cache coherence - 한 스레드가 저장한 값이 다른 스레드에게 '보이는가'를 보장하지 못하는 예시

![Chapter%2078%20%E1%84%80%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B2%20%E1%84%8C%E1%85%AE%E1%86%BC%E1%84%8B%E1%85%B5%E1%86%AB%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%83%E1%85%A9%E1%86%BC%E1%84%80%E1%85%B5%E1%84%92%E1%85%AA%E1%84%92%E1%85%A2%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%2043d7579f10c2464986ff78cc7a277bbb/Untitled.png](Chapter%2078%20%E1%84%80%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B2%20%E1%84%8C%E1%85%AE%E1%86%BC%E1%84%8B%E1%85%B5%E1%86%AB%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%83%E1%85%A9%E1%86%BC%E1%84%80%E1%85%B5%E1%84%92%E1%85%AA%E1%84%92%E1%85%A2%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%2043d7579f10c2464986ff78cc7a277bbb/Untitled.png)

### 잘못된 코드 - 이 프로그램은 얼마나 오래 실행될까?

```java
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
      Thread backgroundThread = new Thread(() -> {
        int i = 0;
        while (!stopRequested) { //loop A
          i++;
        }
      });
      backgroundThread.start();
      TimeUnit.SECONDS.sleep(1);
      stopRequested = true; //loop A가 stop될 거라고 예상되는 시점
    }
  }
```

- 멈추지 않는다 그 이유는 OpenJDK에서 실제 적용하는 기법인 hoisting 기법 때문

→ 동기화를 하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤에나 보게 될지 보증할 수 없다.

```java
//원래 코드
while (!stopRequested) {
  i++;
}

// 최적화한 코드
if (!stopRequested) {
  while (true) {
    i++;
  }
}
```

- 아래처럼 동기화를 하게 되면 언제나 볼 수 있게 보장된다.

→ 쓰기와 읽기 모두가 동기화되어야 동작이 보장된다.

```java
public class StopThread {
  private static boolean stopRequested;

  private static synchronized void requestStop() {
    stopRequested = true;
  }

  private static synchronized boolean stopRequested() {
    return stopRequested;
  }

  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while (!stopRequested()) {
        i++;
      }
    });
    backgroundThread.start();
    TimeUnit.SECONDS.sleep(1);
    requestStop();
  }
}
```

### Volatile 키워드

- volatile 키워드를 사용하면 언제나 메인 메모리로 데이터를 쓰고 읽는다는 뜻

→ synchronized처럼 한 순간에 하나의 스레드가 읽고 쓰는 것을 보장한다는 뜻은 아님.

```java
public class StopThread {
  private static volatile boolean stopRequested;

  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while (!stopRequested) { //loop A
        i++;
      }
    });
    backgroundThread.start();
    TimeUnit.SECONDS.sleep(1);
    stopRequested = true; //loop A가 stop될 거라고 예상되는 시점
  }
}
```

- volatile 키워드를 주의해야 하는 경우
    - 매번 고유한 값을 반환할 의도로 만들어졌다
    - 증가 연산자
        1. 값을 읽고
        2. +1
        3. 새로운 값을 저장
    - 만약 다른 스레드가 이런 사이를 비집고 들어와서, 값을 읽어간다면 첫번째 스레드와 똑같은 값을 돌려받게 된다. 이를 안전 실패(safety failure)라고 한다
        - ex) Thread 1: 0 → 1, Thread 2: 0 → 1

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
	return nextSerialNumber++;
}
```

→ 따라서 위를 해결하려면 volatile 키워드를 빼고, synchronized를 붙여야 한다

```java
private static int nextSerialNumber = 0;

public static synchronized int generateSerialNumber() {
	return nextSerialNumber++;
}
```

### java.util.concurrent.atomic

락 없이도 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨 있음

→ 배타적 실행 지원

```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static int generateSerialNumber() {
	return nextSerialNum.getAndIncrement;
}
```

### 애초에 가변 데이터를 공유하지 말자

- 불변 데이터만 공유하거나 아무것도 공유하지 말자. 다시 말해 가변 데이터는 단일 스레드에서만 쓰도록 하자.
- 한 스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 때 공유하는 부분만 동기화