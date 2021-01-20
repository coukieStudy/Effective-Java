## wait와 notify 보다는 동시성 유틸리티를 애용하라


java.util.concurrent의 고수준 유틸리티는 세 범주로 나눌 수 있다.
1. 실행자 프레임워크
2. 동시성 컬렉션(concurrent collection)
3. 동기화 장치(synchronizer)


#### 동시성 컬렉션

동시성 컬렉션은 List, Queue, Map 과 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 컬렉션이다. 동기화 한 컬렉션(ex - Collections.synchronizedMap)보다 성능적인 측면에서 훨씬 좋다.
동시성 컬렉션에서 동시성을 무력화하는 것은 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.

ConcurrentMap으로 구현한 동시성 정규화 맵

```java
public class Intern {
    private static final ConcurrentMap<String, String> map =
            new ConcurrentHashMap<>();
            
    public static String intern(String s) {
        String result = map.get(s);
        if (result == null) {
            result = map.putIfAbsent(s, s);
            if (result == null)
                result = s;
        }
        return result;
    }
}
```

또 다른 예시로는 Queue 를 확장한 BlockingQueue 가 있다. BlockingQueue.take()는 큐의 첫 원소를 꺼내는데 이 때 큐가 비어있다면 새로운 원소가 추가될 때까지 기다린다. 이러한 특성 때문에 작업 큐로 쓰기에 적절하다. 

작업 큐는 하나 이상의 생산자 스레드가 작업을 큐에 추가하고, 하나 이상의 소비자 스레드가 큐에 있는 작업을 꺼내 처리하는 형태이다.

#### 동기화 장치

동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 해서 서로 작업을 조율할 수 있게 해준다.
CountDownLatch 를 활용해서, 여러 동작들을 동시에 시작해서 모두 완료하기 까지의 시간을 재는 메서드를 작성할 수 있다.

```java
public class ConcurrentTimer {
    public static long time(Executor executor, int concurrency,
                            Runnable action) throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done  = new CountDownLatch(concurrency);

        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                ready.countDown(); // 타이머에 준비를 마쳤음을 알린다.
                try {
                    start.await(); // 모든 작업자 스레드가 준비될 때까지 기다린다.
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    done.countDown();  // 타이머에게 작업을 마쳤음을 알린다.
                }
            });
        }

        ready.await();     // 모든 작업자가 준비될 때까지 기다린다.
        long startNanos = System.nanoTime();
        start.countDown(); // 작업자들을 깨운다.
        done.await();      // 모든 작업자가 일을 마치기를 기다린다.
        return System.nanoTime() - startNanos;
    }
}
```
레거시 코드를 다룰 때 wait / notify 를 써야할 수도 있다.

wait 메서드를 사용할 때에는 반드시 대기 반복문(wait loop) 관용구를 사용하고, 반복문 밖에서는 절대 호출하지 않아야 한다.
```java
synchronized (obj) {
    while (<조건이 충족되지 않았다>)
        obj.wait();
        
    ... // 조건이 충족됐을 때의 동작을 수행한다.
}
```

* 대기 전에 조건을 검사해서 wait를 건너뛰게 하는 것은 응답 불가 상태를 예방하는 조치이다. 만약 조건이 이미 충족되었는데 대기 상태로 빠지면, 그 스레드를 다시 깨울 수 있다고 보장할 수 없다.
* 대기 후에 조건을 검사하여 조건이 충족되지 않았다면 다시 대기하게 하는 것은 안전 실패를 막는 조치이다. 조건이 충족되지 않았는데 스레드가 동작을 이어가면 락이 보호하는 불변식을 깨트릴 수 있다. 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황이 존재한다.
  * 스레드가 notify를 호출한 다음 대기중이던 스레드가 깨어나는 사이에 다른 스레드가 락을 얻어 락이 보호하는 상태를 변경한다.
  * 다른 스레드가 실수로, 혹은 악의적으로 notify를 호출한다. 
  * 깨우는 스레드가 대기 중인 스레드 중 일부만 조건이 충족되어도 notifyAll을 호출해 다른 스레드를 모두 깨운다.
  * 대기 중인 스레드가 notify 없이도 깨어나는 허위 각성 현상이 발생한다.
  
notify / notifyAll 사이에서는 일반적으로 notifyAll 을 사용하는 편이 안전하다. 모든 스레드가 깨어남을 보장해서 정확한 결과를 얻을 수 있다. notify로 최적화 할 수 있지만, notify를 사용하면 관련 없는 스레드가 실수로 혹은 악의적으로 wait를 호출하는 공격으로부터 보호받을 수 있다.







