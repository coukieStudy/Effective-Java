# 인터페이스는 타입을 정의하는 용도로만 사용하라
## 잘못된 예: 상수 인터페이스
- 상수 인터페이스: 메서드 없이 상수에 해당하는 static final 필드로만 구성된 인터페이스
- 상수를 사용할 때 정규화된 이름 (qualified name)을 쓰기 싫어서 인터페이스로 구현하는 경우가 있다.
    - Qualified name: 패키지 이름을 멤버 이름 앞에 붙여 참조하는 것 (ex. `java.lang.Math.PI`;)
    - Simple name: 멤버 이름만으로 참조하는 것 (ex. `PI;`) 
- 상수 인터페이스의 예시
```java
public interface PhysicalConstants {
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    ...
}
```
- 의미상 상수는 내부 구현인데, 인터페이스로 만들면 사실상 내부 구현을 외부 API로 노출하는 것과 같은 짓이다.
- 이렇게 하면 사용자에게 혼란을 주며, 클라이언트 코드가 이 상수들에 종속될 수 있다. 그렇게 되면 나중에 상수를 쓰지 않게 되어도 바이너리 호환성을 위해 인터페이스를 그대로 유지해야 한다.
    - 바이너리 호환성: 어떤 라이브러리의 버전업 과정에서 특정 클래스의 멤버를 삭제하면, 그 라이브러리의 사용자가 업데이트 후 그 멤버를 호출할 때 예외가 발생할 수 있다. 이런 상황을 바이너리 호환성이 깨졌다고 한다.
- 또한 하위 클래스들의 이름공간이 해당 상수들에 침범당한다.
## 상수 인터페이스의 대안
1. 해당 상수를 사용하는 클래스나 인터페이스의 멤버로 추가한다.
    - 예) `Integer`나 `Double`의 `MIN_VALUE`와 `MAX_VALUE`
2. Enum
3. 유틸리티 클래스
```java
pubic class PhysicalConstants {
    private PhysicalConstants() {} // 인스턴스화 방지

    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    ...

// 정적 임포트를 사용하면 simple name으로 호출할 수 있다.
import static PhysicalConstants.*;
}