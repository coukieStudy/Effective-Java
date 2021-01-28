### Item 90 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

Serializable을 구현하기로 결정한 순간, 언어의 정상 메커니즘인 생성자 이외의 방법으로 인스턴스를 생성할 수 있게 된다. 버그와 보안 문제가 일어날 가능성이 커진다는 뜻이다. 직렬화 프록시 패턴(serialization proxy pattern)은 이 위험을 크게 줄여줄 기법이다.

먼저, 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해 private static으로 선언한다. 이 중첩 클래스가 바로 바깥 클래스의 직렬화 프록시다. 중첩 클래스의 생성자는 단 하나여야 하며, 바깥 클래스를 매개변수로 받아야 한다. 이 생성자는 단순히 인수로 넘어온 인스턴스의 데이터를 복사한다. 일관성 검사나 방어적 복사도 필요 없다. 설계상, 직렬화 프록시의 기본 직렬화 형태는 바깥 클래스의 직렬화 형태로 쓰기에 이상적이다. 그리고 바깥 클래스와 직렬화 프록시 모두 Serializable을 구현한다고 선언해야 한다.

Period 클래스를 예로 살 펴보자. 다음은 이 클래스의 직렬화 프록시다. Period는 아주 간단하여 직렬화 프록시도 바깥 클래스와 완전히 같은 필드로 구성되었다.

```java
private static class SerializationProxy implements Serializable {
    private final Date start;
    private final Date end;

    SerializationProxy(Period p) {
        this.start = p.start;
        this.end = p.end;
    }

    private static final long serialVersionUID = 
      234098243823485285L; // 아무 값이나 상관없다. (아이템 87)
}
```

다음으로, 바깥 클래스에 다음의 writeReplace 메서드를 추가한다. 이 메서드 는 범용적이니 직렬화 프록시를 사용하는 모든 클래스에 그대로 복사해 쓰면 된다.

```java
// 직렬화 프록시 패턴용 writeReplace 메서드 
private Object writeReplace(){
        return new SerializationProxy(this);
}
```

이 메서드는 직렬화가 이뤄지기 전에 바깥 클래스의 인스턴스를 직렬화 프록시로 변환해준다.

writeReplace 덕분에 직렬화 시스템은 결코 바깥 클래스의 직렬화된 인스턴스를 생성해낼 수 없다. 하지만 공격자는 불변식을 훼손하고자 이런 시도를 해 볼 수 있다. 다음의 readObject 메서드를 바깥 클래스에 추가하면 이 공격을 가볍게 막아낼 수 있다.

```java
// 직렬화 프록시 패턴용 readObject 메서드
private void readObject(ObjectInputStream stream) throws InvalidObjectException {
    throw new InvalidObjectException("프록시가 필요합니다.");
}
```

마지막으로, 바깥 클래스와 논리적으로 동일한 인스턴스를 반환하는 readResolve 메서드를 SerializationProxy 클래스에 추가한다. 이 메서드는 역직렬화 시에 직렬화 시스템이 직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환하게 해준다.

readResolve 메서드는 공개된 API만을 사용해 바깥 클래스의 인스턴스를 생성하는데, 이 패턴이 아름다운 이유가 바로 여기 있다. 직렬화는 생성자를 이용하지 않고도 인스턴스를 생성하는 기능을 제공하는데, 이 패턴은 직렬화의 이런 언어도단적 특성을 상당 부분 제거한다. 즉, 일반 인스턴스를 만들 때와 똑같은 생성자, 정적 팩터리, 혹은 다른 메서드를 사용해 역직렬화된 인스턴스를 생성하는 것이다. 따라서 역직렬화된 인스턴스가 해당 클래스의 불변식을 만족하는지 검사할 또 다른 수단을 강구하지 않아도 된다. 그 클래스의 정적 팩터리나 생성자가 불변식을 확인해주고 인스턴스 메서드들이 불변식을 잘 지 켜준다면, 따로 더 해줘야 할 일이 없는 것이다.

앞서의 Period.SerializationProxy용 readResolve 메서드는 이렇게 생겼다.

```java
// Period.SerializationProxy용 readResolve 메서드
private Object readResolve() {
    return new Period(start, end); // public 생성자를 사용한다. 
}
```

방어적 복사(코드 88-5)처럼, 직렬화 프록시 패턴은 가짜 바이트 스트림 공격 (코드 88-2)과 내부 필드 탈취 공격(코드 88-4)을 프록시 수준에서 차단해준다. 앞서의 두 접근법과 달리, 직렬화 프록시는 Period의 필드를 final로 선언해도 되므로 Period 클래스를 진정한 불변(아이템 17)으로 만들 수도 있다. 또한 이 리저리 고민할 거리도 거의 없다. 어떤 필드가 기만적인 직렬화 공격의 목표가 될지 고민하지 않아도 되며, 역직렬화 때 유효성 검사를 수행하지 않아도 된다.

직렬화 프록시 패턴이 readObject에서의 방어적 복사보다 강력한 경우가 하나 더 있다. 직렬화 프록시 패턴은 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동한다. 실전에서 무슨 쓸모가 있나 싶겠지만, 쓸모가 있다.

EnumSet의 사례를 생각해보자(아이템 36). 이 클래스는 public 생성자 없이 정적 팩터리들만 제공한다. 클라이언트 입장에서는 이 팩터리들이 EnumSet 인스턴스를 반환하는 걸로 보이지만, 현재의 OpenJDK를 보면 열거 타입의 크기에 따라 두 하위 클래스 중 하나의 인스턴스를 반환한다. 열거 타입의 원소가 64개 이하면 RegularEnumSet을 사용하고, 그보다 크면 JumboEnumSet을 사용하는 것이다.

이제 원소 64개짜리 열거 타입을 가진 EnumSet을 직렬화한 다음 원소 5개 를 추가하고 역직렬화하면 어떤 일이 벌어질지 알아보자. 처음 직렬화된 것은 RegularEnumSet 인스턴스다. 하지만 역직렬화는 JumboEnumSet 인스턴스로 하면 좋을 것이다. 그리고 EnumSet은 직렬화 프록시 패턴을 사용해서 실제로도이렇게 동작한다. 궁금한 분을 위해 EnumSet의 직렬화 프록시 코드를 가져왔다. 정말 간단하지 않은가!

```java
private static class SerializationProxy<E extends Enum<E>> implements Serializable {
    // 이 EnumSet의 원소 타입
    private final Class<E> elementType;
    // 이 EnumSet 안의 원소들
    private final Enum<?>[] elements;

    SerializationProxy(EnumSet<E> set) {
        elementType = set.elementType;
        elements = set.toArray(new Enum<?>[0]);
    }

    private Object readResolve() {
        EnumSet<E> result = EnumSet.noneOf(elementType);
        for (Enum<?> e : elements)
            result.add((E) e);
        return result;
    }

    private static final long serialVersionUID = 362491234563181265L;
}
```

직렬화 프록시 패턴에는 한계가 두 가지 있다. 첫 번째, 클라이언트가 멋대로 확장할 수 있는 클래스(아이템 19)에는 적용할 수 없다. 두 번째, 객체 그래프에 순환이 있는 클래스에도 적용할 수 없다. 이런 객체의 메서드를 직렬화 프록시 의 readResolve 안에서 호출하려 하면 ClassCastException이 발생할 것이다. 직 렬화 프록시만 가졌을 뿐 실제 객체는 아직 만들어진 것이 아니기 때문이다.

마지막으로, 직렬화 프록시 패턴이 주는 강력함과 안전성에도 대가는 따른다. Period 예를 내 컴퓨터에서 실행해보니 방어적 복사 때보다 14%가 느렸다.

**핵심 정리**

제3자가 확장할 수 없는 클래스라면 가능한 한 직렬화 프록시 패턴을 사용하자. 이 패턴 이 아마도 중요한 불변식을 안정적으로 직렬화해주는 가장 쉬운 방법일 것이다.
