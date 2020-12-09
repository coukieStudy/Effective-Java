## 정의하려는 것이 타입이라면, 마커 인터페이스를 사용해라

마커 인터페이스: 아무 메서드도 담고 있지 않고, 단지 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스. ex) Serializable
```java
public interface Serializable {
}
```

### 마커 인터페이스의 장점

1. 마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있다.

    - 마커 애너테이션은 타입이 아니기 때문에 런타임에 발견할 수 있는 오류가 있지만, 마커 인터페이스는 그 자체로 타입이기 때문에 컴파일 시점에 오류를 발견할 수 있다.<br>
    - Serializable은 마커 인터페이스의 장점을 보여줄 수 있었으나 잘 활용하지 못한 예시이다.
    ```java
    // ObjectOutputWriter.writeObject 의 일부
    ...
        else if (obj instanceof Serializable) {
            writeOrdinaryObject(obj, desc, unshared);
        } else {
            if (extendedDebugInfo) {
                throw new NotSerializableException(
                    cl.getName() + "\n" + debugInfoStack.toString());
            } else {
                throw new NotSerializableException(cl.getName());
            }
        }
    ```
    - writeObject()의 parameter가 Serializable 이었으면 컴파일 시점에 오류를 발견할 수 있었을 것이다.
    
2. 마커 인터페이스는 적용 대상을 더 정밀하게 지정할 수 있다.
    - 적용 대상(@Target)을 ElementType.TYPE으로 선언한 애너테이션 은 모든 타입(클래스, 인터페이스, 열거 타입, 애너테이션)에 달 수 있다. 부착 할 수 있는 타입을 더 세밀하게 제한하지는 못한다는 뜻이다. 
    - 그런데 특정 인 터페이스를 구현한 클래스에만 적용하고 싶은 마커가 있을 경우 그냥 마킹하고 싶은 클래스에서만 그 인터페이스를 구현하면 된다.
    - Set 은 일종의 마커 인터페이스라고 볼 수 있다. 실제로 Set은 Collection에 존재하는 메서드들만 존재한다. 다만 중복된 원소가 존재하지 않는다는 제약이 있기 때문에 엄밀하게 마커 인터페이스라고 보기에는 어려울 수도 있다.
    - 그런데 이런 마킹을 Collection의 하위 타입에만 적용하고 싶다면, 마커 애너테이션으로는 적용 대상을 지정할 수 없지만, 마커 인터페이스로는 확장을 하면 된다.(```public interface Set<E> extends Collection<E>```)
    
    
### 마커 애너테이션의 장점

- 마커 애너테이션이 마커 인터페이스보다 나은 점으로는 거대한 애너 테이션 시스템의 지원을 받는다는 점을 들 수 있다. 따라서 애너테이션을 적극 활용하는 프레임워크에서는 마커 애너테이션을 쓰는 쪽이 일관성을 지키는 데 유리할 것이다..

### 마커 애너테이션과 인터페이스의 선택

- 클래스와 인터페이스 외의 프로그램 요소(모듈, 패키 지, 필드, 지역변수 등)에 마킹해야 할 때 애너테이션을 쓸 수밖에 없다. 마커 인터페이스는 클래스와 인터페이스에만 적용될 수 있다.

- 마커를 클래스나 인터페이스에 적용할 때, "이 마킹이 된 객체를 매개변수 로 받는 메서드를 작성할 일이 있을까?"에 대한 대답이 YES 이면 마커 인터페이스, NO 이면 마커 애너테이션
