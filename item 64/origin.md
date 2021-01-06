# 객체는 인터페이스를 사용해 참조하라.

> 적합한 인터페이스만 있다면 매개변수뿐 아니라 반환값, 변수, 필드를 전부 인터페이스 타입으로 선언하라. 그렇지 않다면 당연히 클래스를 참조해야 한다.

### 인터페이스를 사용하는 경우

```java
// ex) use class
LinkedHashSet<Son> sonSet = new LinkedHashSet<>();

// ex) use interface
Set<Son> sonSet = new LinkedHashSet<>();
Set<Son> sonSet = new HashSet<>();
```

**장점**: 구현 클래스를 교체할 때. ex) 속도향상을 위해 HashMap -> EnumMap

**주의**: 원래의 클래스가 인터페이스의 일반 규약 외의 특별한 기능을 제공하고, 주변 코드가 이 기능에 기대어 동작할 경우.

### 인터페이스를 사용하지 않는 경우

1. 값 클래스. ex) String, BigInteger
2. 클래스 기반으로 작성된 프레임워크가 제공하는 객체들. ex) OutputStream in java.io
3. 인터페이스에는 없는 특별한 메서드를 제공하는 클래스들. ex) PriorityQueue class Vs Queue interface
