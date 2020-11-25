# 34. int 상수 대신 열거 타입을 사용하라

## 정수 열거 패턴 (int enum pattern)

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPEN = 1;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
```

타입 안전을 보장할 수 없다. 따라서 오렌지를 파라미터로하는 메서드에 사과를 보내도 컴파일러는 경고 메세지 출력하지 않는다.

표현력이 좋지 않다. 별도 namespace 지원하지 않기 때문에 접두사를 사용할 수 밖에 없다.

프로그램이 깨지기 쉽다. 값이 그대로 새겨지고 컴파일을 다시 하지 않으면 엉뚱하게 동작할 수 있다.
```java
public static final int APPLE_PIE = 0;

System.out.println(APPLIE_PIE); // 아래와 같은 코드!
System.out.println(0);
```
정수 상수는 문자열로 출력하기 까다롭다. 
```java
public static final int APPLE_PIE = 0;
System.out.println(APPLIE_PIE); // APPLIE_PIE 가 아닌 0이 출력!
```

같은 정수 열거 그룹에 속한 상수만 순회하고 싶을 때, 그 안에 속한 상수가 몇 개인지 알고 싶을 때 방법이 마땅치 않다.

## 문자열 열거 패턴 (String enum pattern)

상수의 의미를 출력할 수 있지만 문자열 비교에 따라 성능이 저하될 수 있다.

## 열거 타입 (Enum type)

> 열거 타입 자체는 클래스, 상수 하나당 자신의 인스턴스를 만들어 public static final 필드로 공개한다.
열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final이다.

### 열거 타입의 장점

- 열거 타입 선언으로 만들어진 인스턴는 딱 하나다.
- 열거 타입은 컴파일타임 타입 안정성을 제공한다.
- 열거 타입은 각자의 namespace가 있다.
- 열거 타입의 toString 메소드는 출력하기에 적합한 문자열을 내어준다. 상수 이름을 문자열로 반환한다.
- 열거 타입에는 임의의 메소드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다. 또한 Enum 자체는 Object 메소드들과 Comparable, Serializable을 구현해두었다.
```java
package java.lang;
public abstract class Enum<E extends Enum<E>> implements Comparable<E>, Serializable {...}
```
- 열거 타입 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드 values()를 제공한다. 
- valueOf(EnumType): 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환한다.
- 열거 타입에 선언한 상수 하나를 제거하더라도 제거한 상수를 참조하지 않는 클라이언트에는 아무 영향이 없다. 또한 그런 상수를 참조를 하더라도 컴파일 에러가 발생할 것이다.

### 데이터나 메서드를 갖는 열거 타입

- 특정 데이터와 열거 타입 상수를 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장한다.
- private final로 필드를 선언해야한다.

```java
public enum Planet {
    MERCURY(3.302e+23,2.439e6),
    VENUS(4.869e+24,6.052e6),
    EARTH(5.975e+24, 6.378e6);
    
    private final double mass;
    private final double radius;
    private final double surfaceGravity;

    private static final double G = 6.67300E-11;

    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        this.surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() { return mass; }

    public double radius() { return radius; }

    public double surfaceGravity() { return surfaceGravity; }
    
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;
    }
}
```

### 상수별 메서드 구현

```java
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE
}
```

**switch문**으로 작성시, 새로운 상수 추가할 때마다 case문을 추가해야한다.

```java
public enum Operation {
    PLUS, MINUS;
    
    public double apply(double x, double y) {
        switch(this) {
            case PLUS: return x + y;
            case MINUS: return x - y;
        }
        throw new AssertionError("알 수 없는 연산: " + this);
    }
}
```

**abstract method**를 사용하면 재정의하지 않으면 컴파일 오류로 알려준다.

```java
public enum Operation {
    PLUS{ public double apply(double x, double y) { return x + y; }},
    MINUS{ public double apply(double x, double y) { return x - y; }};
    
    public abstract double apply(double x, double y);    
}
```

### 상수로 열거 타입 구하기

```java
private static final Map<String, OperationWithToString> stringToEnum = 
	Stream.of(values())
	.collect(Collectors.toMap(Objects::toString, e -> e));
    
public static Optional<OperationWithToString> fromString(String symbol){
	return Optional.ofNullable(stringToEnum.get(symbol));
}
```

### 전략 열거 타입 패턴

상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다.

요일별로 일당을 계산하는 열거타입의 경우, 아래처럼 전략(PayType)을 선택하게 하면 깔끔하다.

```java
enum PayrollDay {
    MONDAY(WEEKDAY),
    TUESDAY(WEEKDAY),
    WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY),
    FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND),
    SUNDAY(WEEKEND);
    
    private final PayType payType;
    
    PayrollDay(PayType payType) {
        this.payType = payType;
    }
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }
    
    enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        
        WEEKEND {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked * payRate / 2
            }
        };
        
        abstract int overtimePay(int minutesWorked, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;
        
        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked & payRate;
            return basePay + overtimePay(minutesWorked, payRate);
        }
    }
}
```

아래처럼 기존의 열거 타입의 상수별 동작을 혼합해 넣을 때에는 추가 구현이 필요없는 switch문이 좋은 선택이 될 수 있다. 

```java
public static Operation inverse(Operation operation) {
        switch (operation) {
            case PLUS:
                return Operation.MINUS;
            case MINUS:
                return Operation.PLUS;
            case TIMES:
                return Operation.DIVDE;
            case DIVDE:
                return Operation.TIMES;
        }
        throw new AssertionError("알 수 없는 연산 : " +operation);
}
```

### 열거타입의 성능과 사용법

- 열거 타입을 메모리에 올리는 공간, 초기화 시간 늘어나지만 미미.
- 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자!
- 열거 타입에 정의된 상수 개수가 영원히 고정, 불변일 필요는 없다.
