## Item 1 생성자 대신 정적 팩터리 메서드를 고려하라

### 정적 팩터리 메서드란?

- public 생성자와 다르게 아래와 같이 클래스의 인스턴스를 반환하는 정적 메서드를 뜻한다.
- 아래는 Boolean의 valueOf 예시
```java
public static Boolean valueOf(boolean b){
  return b ? Boolean.TRUE: Boolean.FALSE;
}
```

- 정적 팩터리 메서드는 다음 링크에 있는 팩터리 메서드 패턴과는 다르다. (이름도 비슷하고 둘 다 객체 생성에 관여하기 때문에 둘의 차이점을 알아두는 것 중요할 듯)
  - [디자인 패턴 - Factory Method Pattern1](https://jdm.kr/blog/180)
  - [디자인 패턴 - Factory Method Pattern2](https://beomseok95.tistory.com/244)
  
### 정적 팩터리 메서드의 장점
1. 이름을 가질 수 있다.
    - 가독성이 상승: BigInteger 생성자 vs BigInteger.probablePrime
    - 생성자의 경우 하나의 시그니처(같은 매개변수)로 생성자를 하나만 만들 수 있다. 하지만 정적 팩터리 메서드는 여러 가지 가능
      - 이를 피하는 방법은 입력 매개변수의 순서를 변경하는 것인데, Anti Pattern
      
2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
    - 생성자는 return 값이 객체인데, 정적 팩터리 메서드는 마음대로 할 수 있기에
    - 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.
      - 반복되는 요청이 있는 상황에서 성능을 높일 수 있다.
    - <b>인스턴스 통제 클래스</b>: 언제 어느 인스턴스를 살아 있게 할지를 철저하게 통제하는 클래스
      * [싱글톤 패턴](https://github.com/HaeUlNam/TIL/blob/master/DesignPattern/Singleton.md), 인스턴스화 불가(Item 4), 불변 값 클래스(Item 17)에서 동치인 인스턴스가 단 하나임을 보장 가능
      * 플라이웨이트 패턴에 근간이 되며, 열거 타입(Item 34)은 인스턴스가 하나만 만들어짐을 보장한다.
    - 예시: Boolean.valueOf(boolean)
    ```java
    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);
    
    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
    //참고: http://hg.openjdk.java.net/jdk8u/jdk8u/jdk/file/dc4322602480/src/share/classes/java/lang/Boolean.java
    ```
    
 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
    - 반환할 객체의 클래스를 자유롭게 선택할 수 있는 '엄청난 유연성'을 선물
      - API를 만들 때 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지 가능
      - TODO) 이는 인터페이스를 정적 팩토리 메서드의 반환 타입으로 사용하는 인터페이스 기반 프레임워크(Item 20)를 만드는 핵심 기술이기도 하다.
    - Java 8 전까지는 Interface에 정적 메서드를 선언할 수 없었다. 그렇기 때문에 이름이 "Type"인 인터페이스를 반환하는 정적 메서드가 필요하면, "Types"라는 (인스턴스화 불가인) 동반 클래스를 만들어
    그 안에 정의하는 것이 관례였다.
      - Collection은 Collections라는 동반 클래스를 통해 유용한 기능들을 제공한다.
      - java.util.Collections: 45개의 유틸리티 구현체 대부분을 Collections를 통해 얻게 된다. 45개 대부분을 공개하지 않기 때문에 API의 외견을 훨씬 작게 만들 수 있었다.
    ```java
    public static <K,V> Map<K,V> unmodifiableMap(Map<? extends K, ? extends V> m) {
        return new UnmodifiableMap<>(m);
    }
    
    private static class UnmodifiableMap<K,V> implements Map<K,V>, Serializable {
        //여러 가지 함수들
    }
    //UnmodifiableMap 개념: https://lng1982.tistory.com/155
    ```
    
    - 나아가 정적 팩터리 메서드를 사용하는 클라이언트는 얻은 객체를 인터페이스만으로 다루게 된다.(아이템 64) -> Best Pratice
    - Java 8부터는 인터페이스가 default 키워드를 이용해서 정적 메서드를 구현할 수 있기에 동반 클래스가 필요 없다.
      - 하지만 public 정적 메서드만 사용할 수 있기에 private을 두려면 private-package나 Java 9를 사용해야 한다.
      - [Java 8 - Interface의 변화](http://happinessoncode.com/2017/04/19/java8-changes-in-interface/)

4. 입력 매개 변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
    - 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.
    - 예시) EnumSet 클래스는 public 생성자 없이 오직 정적 팩터리만 제공하는데, 원소의 수에 따라 두 가지 하위 클래스 중 하나 return.
      - 64이하면 원소들을 long 변수 하나로 관리하는 RegularEnumSet
      - 65이상이면 long 배열로 관리하는 JumboEnumSet의 인스턴스를 반환
      ```java
      //noneOf 메소드에서 상황에 따라 다른 구현체 객체들을 만들어서 반환해주고 있다.
      public static noneOf(...) {
          if (enumElementSize <= 64)
              return new RegularEnumSet<>(...);  
          else
              return new JumboEnumSet<>(...);
      }
      ```
    - 위와 같이 진행했을 때 장점
      - Client는 어떤 구현체를 사용하고 있는지 몰라도 된다: 더 개선한 세 번째, 네 번째 클래스를 반영하더라도 client 코드에는 변경이 없다.

5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
    - 이런 유연함은 서비스 제공자 프레임워크를 만드는 근간이 된다. 대표적인 예시로는 JDBC
      - 여기서 제공자(provider)는 서비스의 구현체다. 그리고 이 구현체들을 클라이언트에 제공하는 역할을 프레임워크가 통제하여, 클라이언트를 구현체로부터 분리해준다.
      - 서비스 제공자 프레임워크 3개의 핵심 컴포넌트
        - Service Interface: 서비스의 동작을 정의한다.
        - Provider Registeration API: 제공자가 구현체를 등록할 때 사용한다.
        - Service Access API: 클라이언트가 서비스의 인스턴스를 얻을 때 사용한다.
      - Service provider Interface(위 3개와 종종 같이 사용됨): 서비스 인터페이스의 인스턴스를 생성하는 팩터리 객체를 설명해준다. 서비스 제공자 인터페이스가 없다면 각 구현체를 인스턴스로 만들 때 리플레션(Item 65)을 사용해야 한다. (TODO) Why??)
    - JDBC로 표현하면
      - Service Interface: Connection
      - Provider Registeration API: DriverManager.registerDriver
      - Service Access API: DriverManager.getConnection
      - Service provider Interface: Driver
    - 서비스 제공자 프레임워크 패턴에는 여러 변형이 존재
      - Bridge Pattern//TODO
      - DI 프레임워크//TODO
      - 자바 5부터는 ServiceLoader가 제공되어 프레임워크를 직접 만들 필요가 없어졌다. JDBC는 자바 5전에 생긴 개념.
 
### 정적 팩토리 메서드의 단점
1. 상속을 하려면 public이나 protexted 생성자가 필요하니 정적 팩터리 매서드만 제공하면 하위 클래스를 만들 수 없다.
    - 앞서 이야기한 프레임워크의 유틸리티 구현 클래스들은 상속할 수 없다는 이야기
    - 상속보다 컴포지션을 사용(Item 18)하도록 유도하고 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서 오히려 장점으로 받아들일 수도 있다.

2. 정적 팩토리 메서드는 프로그래머가 찾기 어렵다.
    - 따라서 아래의 정적 팩토리 메서드 명명 규칙을 알면, 코드 속에서 더 빠르게 정적 팩토리 메서드를 찾을 수 있다.
    - 아래에서 나오는 Type은 팩토리 메서드가 반환할 객체의 타입이다.
      |함수명|내용|예시|
      |------|---|---|
      |from|매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드|Data d = Date.from(instant);|
      |of|여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드|Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);|
      |valueOf|from과 of의 더 자세한 버전|BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);|
      |instance or getInstance|매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다|StackWalker luke = StackWaler.getInstance(options);|
      |create or newInstance|instance혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다|Object newArray = Array.newInstance(classObject, arrayLen);|
      |getType|getInstance와 같으나 생성할 클래스가 아닌 다른 클래스에 팩터리 매서드를 정의할 때 사용|FileStore fs = Files.getFileStore(path);|
      |newType|newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 매서드를 정의할 때 사용|BufferedReader br = Files.newBufferedReader(path);|
      |type|getType와 newType의 간결한 버전|List<Complaint> litany = Collections.list(legacyLitany);|
      

### 핵심 정리
- 정적 팩토리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것은 좋다. 그렇다고 하더라도 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를
제공하던 습관이 있다면 고치자.
    
### 참고
- [EnumSet이 new 연산자를 사용하지 않는 이유](https://siyoon210.tistory.com/152)
- [How do I decrypt "Enum<E extends Enum<E>>"?](http://www.angelikalanger.com/GenericsFAQ/FAQSections/TypeParameters.html#FAQ106)
- [Java Enum 활용기](https://woowabros.github.io/tools/2017/07/10/java-enum-uses.html)
