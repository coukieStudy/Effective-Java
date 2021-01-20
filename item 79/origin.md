## 과도한 동기화는 피하라

다음과 예제는 Set에 원소가 추가될 때마다 관찰자들에게 notify 하는 기능을 가진 클래스이다.

```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) { super(set); }

    private final List<SetObserver<E>> observers
            = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
       }
    }
    
    private void notifyElementAdded(E element) {
        synchronized(observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }

    @Override public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element);  // Calls notifyElementAdded
        return result;
    }
}

@FunctionalInterface public interface SetObserver<E> {
    void added(ObservableSet<E> set, E element);
}
```

다음은 0부터 99까지를 출력하는 예제이다. 잘 동작함

```java
public class Test1 {
    public static void main(String[] args) {
        ObservableSet<Integer> set =
                new ObservableSet<>(new HashSet<>());

        set.addObserver((s, e) -> System.out.println(e));

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```

평상시에는 집합에 추가된 정수값을 출력하다가, 그 값이 23이면 자기 자신을 제거하는 관찰자를 추가
```java
public class Test2 {
    public static void main(String[] args) {
        ObservableSet<Integer> set =
                new ObservableSet<>(new HashSet<>());

        set.addObserver(new SetObserver<>() {
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if (e == 23)
                    s.removeObserver(this);
            }
        });

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```
그런데 위의 예제를 실행하면 ConcurrentModificationException 이 발생한다. 

메서드 호출 순서: add -> notifyElementAdded -> added -> removeObserver -> remove.

즉 리스트를 순회하는 도중에 해당 원소를 제거하려 하기 때문에 허용되지 않은 동작인 것이다. notifyElementAdded 메서드는 동기화 블록 안에 있으므로 동시 수정이 일어나지는 않지만, 정작 자신이 콜백을 거쳐 되돌아와서 수정을 하는 것 까지는 막지 못한다.

이번에는 원소 제거를 ExecutorService를 이용해서 시도.

```java
public class Test3 {
    public static void main(String[] args) {
        ObservableSet<Integer> set =
                new ObservableSet<>(new HashSet<>());
                
        set.addObserver(new SetObserver<>() {
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if (e == 23) {
                    ExecutorService exec =
                            Executors.newSingleThreadExecutor();
                    try {
                        exec.submit(() -> s.removeObserver(this)).get();
                    } catch (ExecutionException | InterruptedException ex) {
                        throw new AssertionError(ex);
                    } finally {
                        exec.shutdown();
                    }
                }
            }
        });

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```
이 프로그램은 교착상태에 빠진다. 

백그라운드 스레드가 s.removeObserver 를 호출하면 observer는 이미 메인 스레드가 notifyElementAdded 락을 쥐고 있는 상태. 한편 메인스레드에서는 백그라운드 스레드가 removeObserver를 완료하기를 기다리고 있는 상태라서 교착이 발생한다.

위의 상황에서 added가 호출될 때 동기화 영역이 보호하는 자원(observers)은 동일했다. 만약 불변식이 임시로 깨진 경우라면 어떻게 되는가? 

> 자바 언어의 락은 재진입(reentrant)을 허용하므로 교착상태에 빠지지는 않는다. 예외를 발생시킨 첫 번째 예에서라면 외계인 메서드를 호출하는 스레드는 이미 락을 쥐고 있으므로 다음번 락 획득도 성공한다. 그 락이 보호하는 데이터에 대해 개념적으로 관련이 없는 다른 작업이 진행 중인데도 말이다. 이것 때문에 실로 참혹한 결과가 빚어질 수도 있다. 문제의 주 원인은 락이 제 구실을 하지 못했기 때문이다. 재진입 가능 락은 객체 지향 멀티스레드 프로그램을 쉽게 구현할 수 있도록 해주지만, 응답 불가(교착상태)가 될 상황을 안전 실패(데이터 훼손)로 변모시킬 수도 있다.

이런 상황은 외계인 메서드 호출을 동기화 블록 바깥으로 옮기면 된다. notifyElementAdded 에서라면, 관찰자 리스트를 복사해서 쓰면 락 없이도 안전하게 순회할 수 있다. 이 방식을 통해 예외 발생과 교착 증상을 해결할 수 있다.

```java
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot)
        observer.added(this, element);
}
```

자바의 동시성 컬렉션 라이브러리의 CopyOnWriteArrayList는 내부를 변경하는 작업은 항상 복사본을 만들어 수행하도록 구현되어 있다. 따라서 이를 활용하면 명시적인 동기화 없이 사용할 수 있다.
```java
private final List<SetObserver<E>> observers =
        new CopyOnWriteArrayList<>();
        
public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers)
        observer.added(this, element);
}
```

동기화 영역에서는 가능한 일을 적게 하는 것이 핵심이다. 가변 클래스를 작성할 때에는
1. 동기활르 전혀 하지 말고, 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 동기화를 하게 두거나
2. 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들어야 한다.
단 두 번째 방법은 동시성을 월등히 개선할 수 있을 때에만 선택해야 한다. 
