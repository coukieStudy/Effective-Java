# private 생성자나 열거 타입으로 싱글턴임을 보장하라

### 싱글턴이란
* 인스턴스를 오직 하나만 생성할 수 있는 클래스
* 클래스를 싱글턴으로 만들면 사용하는 클라이언트가 테스트하기 어려울 수도 있다(싱글턴 인스턴스를 mock구현으로 대체할 수 없기 때문)

### 싱글턴을 만드는 두 방법
공통적으로 1) 생성자는 private으로 설정하여 임의의 객체 생성을 방지하며 2) 생성된 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 제공한다. 이 수단이 final field인지, 정적 팩터리 메서드인지에 따라 싱글턴 생성 방식이 달라진다.

1) public static final field 방식의 싱글턴
```{.java} 
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {...}
    public void leaveTheBuilding() {...}
}
```

* private 생성자는 public static final field 인 Elvis.Instance 를 초기화할 때 한번만 호출된다.
* 이 방식의 장점은 해당 클래스가 싱글턴임이 API에 명확하게 드러난다는 점과, 코드 작성이 간결하다는 점이다.

2) 정적 팩터리 방식의 싱글턴
```{.java}  
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {...}
    public static Elvis getInstance() {
      return INSTANCE;
    }
    public void leaveTheBuilding() {...}
}
```

정적 팩터리 방식의 장점은 다음과 같다
* API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다. (예를들어 스레드별로 다른 인스턴스를 반환하도록 하게 할 수 있다) -> 어떻게...?
	* 만약 다음과 같은 코드가 있다면, 해당 객체는 스레드 별로 다른 인스턴스를 반환할 것인가?
```{.java}  
public class Elvis {
    private static final Elvis INSTANCE = null;
    private Elvis() {...}
    public static Elvis getInstance() {
      if (INSTANCE == null) {
        return new Elvis();
      }
      return INSTANCE;
    }
    public void leaveTheBuilding() {...}
}
```
* 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 바꿀 수 있다. 
~~~
//GenericSingletonFactory.java

interface applySame<T> {  
    T apply(T arg);  
}  
  
public class GenericSingletonFactory<E> {  
    private static applySame<Object> IDENTITY_FUNCTION = (t) -> t;  
    @SuppressWarnings("unchecked")  
    public static <E> applySame<E> getIdentityFunction() {  
      return (applySame<E>) IDENTITY_FUNCTION;  
    }
}

//Main.java

public class Main {  
    public static void main(String[] args) {  
        applySame<String> stringApplySame = GenericSingletonFactory.getIdentityFunction();  
        applySame<Integer> integerApplySame = GenericSingletonFactory.getIdentityFunction();  
        System.out.println(stringApplySame.apply("XYZ"));  
        System.out.println(stringApplySame.hashCode());  
        System.out.println(integerApplySame.hashCode());  
    }
}
~~~
* 위의 코드에서 stringApplySame 과 integerApplySame 가 같은 객체임을 알 수 있다.
	* 그렇다면 다음 코드는...?
	
~~~
// GenericSingletonFactory.java 
interface applySame<T> {  
    T apply(T arg);  
}  
  
public class GenericSingletonFactory<E> {  
    public static applySame<Object> IDENTITY_FUNCTION = (t) -> t;  
}

//Main.java
public class Main {  
    public static void main(String[] args) {  
        applySame<String> stringApplySame = (applySame<String>) GenericSingletonFactory.IDENTITY_FUNCTION;  
        applySame<Integer> integerApplySame = (applySame<Integer>) GenericSingletonFactory.IDENTITY_FUNCTION;  
        System.out.println(stringApplySame.hashCode());  
        System.out.println(integerApplySame.hashCode());  
    }
}
~~~

* 정적 펙터리의 메서드 참조를 Supplier 로 사용할 수 있다. 예를 들어 `Supplier<Elvis> supplier = Elvis::getInstance` 로 사용할 수 있다. 

위 두 방식 모두 Reflection API 를 통해 private 생성자에 접근할 수 있다면, 싱글턴의 객체 유일성이 보장되지 않는다. 따라서 다음과 같은 코드를 통해 객체의 중복 생성을 방지할 수 있다. 
```{.java}  
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {...}
    public static Elvis getInstance() throws Exception {
      if (INSTANCE != null) {
        throw new Exception();
      }
      return INSTANCE;
    }
    public void leaveTheBuilding() {...}
}
```

또한 구현에 따라 멀티스레드 환경에서는 싱글턴이 깨질수도 있다. 참고자료: [https://medium.com/@joongwon/multi-thread-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C%EC%9D%98-%EC%98%AC%EB%B0%94%EB%A5%B8-singleton-578d9511fd42](https://medium.com/@joongwon/multi-thread-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C%EC%9D%98-%EC%98%AC%EB%B0%94%EB%A5%B8-singleton-578d9511fd42)

마지막으로, 직렬화의 경우 역직렬화의 과정에서 싱글턴의 객체유일성이 보장되지 않을 수 있다. 이 때에는 모든 인스턴스 필드를 transient 로 선언하고 readResolve 메서드를 구현해야 한다. (readResolve 의 구현은 getInstance()와 똑같이 구현하면 된다)

### Enum 타입의 싱글턴

원소가 하나인 Enum 타입을 선언하면 싱글턴을 만들 수 있다. 직렬화나 리플렉션 공격에서도 객체의 유일성을 보장할 수 있는 방법이다. 많은 상황에서 원소가 하나인 Enum 타입이 싱글턴을 만드는 가장 좋은 방법이 될 수 있으나, 만드려는 객체가 Enum 이외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.
