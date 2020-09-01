## 다 쓴 객체 참조를 해제하라

- C/C++ 같은 언어에 비해 GC가 존재하기 때문에 메모리 관리에 더 이상 신경쓰지 않아도 된다고 오해할 수 있는데, 절대 사실이 아님!
  * '메모리 누수'부분을 신경써야 한다.
  * [고급] GC 알고리즘을 이해하고 시스템에 맞는 튜닝 필요.
- 메모리 누수는 왜 신경써야 할까?
  * 메모리 누수가 계속 발생하게 되면 GC 활동과 메모리 사용량이 점차 늘어나 결국 성능이 저하
  * 심한 경우 디스크 페이징이나 OOM(OutOfMemoryError)를 일으켜 프로그램이 예기치 않게 종료되기도 한다.
- 아래 코드에서 메모리 누수 부분은 어디일까?
  * pop부분이다. --size를 하게 되면 앞으로 해당 부분은 더 이상 사용하지 않게 되는데, 참조는 여전히 하고 있으니 GC 대상이 아니게 된다.
  * size를 넘어가게 되면 ensureCapacity 알고리즘에 의해, 새로운 크기의 객체를 할당하게 된다.
```java
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY]; 
  }
  public void push(Object e) { 
    ensureCapacity();
    elements[size++] = e; 
  }
  public Object pop() { 
    if (size == 0)
      throw new EmptyStackException(); 
    return elements[--size];
  }
  /**
  * 원소를 위한 공간을 적어도 하나 이상 확보한다.
  * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다. */
  private void ensureCapacity() { 
    if (elements.length == size)
      elements = Arrays.copyOf(elements, 2 * size + 1);  
  }
}
```

- 해결책은?
  * null 처리하여 gc에 대상이 되도록 하자. 계속 참조가 되고 있으면 gc 대상이 영원히 아니게 된다.
  * 아래는 실제 Java Stack 코드이다. Stack은 Vector를 상속받아서 사용하고 있는데 vector에 removeElementAt 함수에 null 표시하는 것이 처리되어 있다.
  ```java
  elementCount--;
  elementData[elementCount] = null; /* to let gc do its work */
  ```
  * 다 쓴 참조를 null처리하면 다른 이점도 따라온다. 만약 null처리한 참조를 실수로 사용하려 하면 프로그램은 즉시 NPE(NullPointerException)을 던지면 종료된다. 프로그램 오류는 가능한 한 조기에 발견하는 것이 
  좋다.
  * 하지만 모든 객체를 쓰고나자마자 null처리할 필요는 없다. 그러면 코드만 지저분해진다.. 따라서 변수의 범위를 최소가 되게 정의하는 것이 가장 좋은 메모리 해제 방법이다. (지역변수는 괄호가 끝난 후에 메모리가 해제되기 때문이다.)
  

### 메모리 누수의 주범들

- 자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다.
  * 원소를 다 사용한 즉시 그 원소가 참조한 객체들을 다 null 처리
  * scope을 최소로 하는 것이 더 좋다.
- 캐시 역시 메모리 누수를 일으키는 주범이다.
  * 객체 참조를 캐시에 넣고 나서, 이 사실을 까맣게 잊은 채 그 객체를 다 쓴 뒤로도 한참을 그냥 놔두는 일을 자주 접할 수 있다.
  * 해결책: 캐시 외부에서 키를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요한 상황이라면 WeakHashMap을 사용해 캐시를 만들자. 다 쓴 엔트리는 그 즉시 자동으로 제거될 것이다.
    - [Java – Collection – Map – WeakHashMap (약한 참조 해시맵)](http://blog.breakingthat.com/2018/08/26/java-collection-map-weakhashmap/)
    - [WeakHashMap Test](https://github.com/coukieStudy/Effective-Java/blob/master/item%207/WeakHashMap%20Test.md)
  * 캐시를 만들 때 보통은 캐시 엔트리의 유효 기간을 정확히 정의하기 어렵기 때문에 시간이 지날수록 엔트리의 가치를 떨어뜨리는 방식을 흔히 사용한다. 이런 방식에서는 쓰지 않는 엔트리를 이따금 청소해줘야 한다.
    - Scheduled ThreadPoolExecutor 같은 백그라운드 스레드를 활용: 정해진 시간 이후에 cache clean
    - 캐시에 새 엔트리를 추가할 때 부수 작업으로 수행하는 방법
      * [LinkedHashMap의 removeEldestEntry](https://www.geeksforgeeks.org/linkedhashmap-removeeldestentry-method-in-java/)    
- 리스너 혹은 콜백
  * 클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는다면, 뭔가 조치해주지 않는 한 콜백은 계속 쌓여갈 것이다. 이럴 때 콜백을 약한 참조(Weak reference)로 저장하면 가비지 컬렉터가 즉시 수거해간다.
  * Android - BroadCast Receiver: 화면이 끝나는 순간 해제를 안하면 memory leak

### 핵심 정리
- 메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다. 이런 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 한다. 그래서 이런 종류의 문제는 예방법을 
익혀두는 것이 매우 중요하다.

### 추가적인 Dive Deep
- 힙 프로파일러: https://imasoftwareengineer.tistory.com/4
- GC 알고리즘
 * https://d2.naver.com/helloworld/1329
 * https://stackoverflow.com/questions/33206313/default-garbage-collector-for-java-8
- 디스크 페이징
- 왜 배열 크기를 두 배씩 늘릴까?
- 성능 때문에 매번 gc에 대상이 되도록 하는 것이 좋은 방법은 아닌거 같은데, 위처럼 pop했던 메모리를 재활용하는 방법은 없을까?
  * ArrayList등에 보면 size와 capacity를 나눠서 사용한다.
  * size가 실제 크기, capacity가 할당된 메모리 크기
