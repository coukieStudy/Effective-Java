# Chapter 38 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라.

## [TypeSafe Enum Pattern vs Enum type](https://stackoverflow.com/questions/5092015/advantages-of-javas-enum-over-the-old-typesafe-enum-pattern)

- **TypeSafe Enum Pattern은 무엇이지?** (위 링크 참고)
- **enum은 확장할 수 없지만, TypeSafe Enum Pattern은 가능하다.**
    - 왜 확장할 수 없나? (참고: [Baeldung - Extending Enums in Java](https://www.baeldung.com/java-extending-enums))
        - 아래 BasicStringOperation을 disassemble한 것이 하단의 결과이다.
        - final 클래스이니깐 자식 클래스가 있을 수 없고, Enum을 extend하고 있으니 다른 클래스를 상속할 수 없다.

    ```java
    //origin enum
    public enum BasicStringOperation {
        TRIM("Removing leading and trailing spaces."),
        TO_UPPER("Changing all characters into upper case."),
        REVERSE("Reversing the given string.");
     
        private String description;
     
        // constructor and getter
    }

    //disassemble
    $ javap BasicStringOperation  
    public final class com.baeldung.enums.extendenum.BasicStringOperation 
        extends java.lang.Enum<com.baeldung.enums.extendenum.BasicStringOperation> {
      public static final com.baeldung.enums.extendenum.BasicStringOperation TRIM;
      public static final com.baeldung.enums.extendenum.BasicStringOperation TO_UPPER;
      public static final com.baeldung.enums.extendenum.BasicStringOperation REVERSE;
     ...
    }
    ```

    - 그럼 확장할 수 없는 게 나쁜 건가?
        - 그렇지 않다. 대부분의 경우에는 열거 타입을 확장하는 것은 좋지 않은 생각이다. 의미가 불명확해질 수 있다. 또한, 확장을 고려하면 설계와 구현이 복잡해지는 단점도 있다.

### 열거 타입을 확장하는 예시

- API가 제공하는 기본 연산 외에 사용자 확장 연산을 추가할 수 있도록 열어줘야 하는 경우가 있다.

## Interface를 이용한 열거 타입 확장

- 연산 코드용 interface를 정의하고 열거 타입이 이 인터페이스를 구현하게 하면 된다.

```java
public interface Operation {
  double apply(double x, double y);
}

public enum BasicOperation implements Operation {
  PLUS("+") {
    public double apply(double x, double y) {
      return x + y;
    }
  },
  MINUS("-") {
    public double apply(double x, double y) {
      return x - y;
    }
  },
  TIMES("*") {
    public double apply(double x, double y) {
      return x * y;
    }
  },
  DIVIDE("/") {
    public double apply(double x, double y) {
      return x / y;
    }
  };
  private final String symbol;

  BasicOperation(String symbol) {
    this.symbol = symbol;
  }

  @Override
  public String toString() {
    return symbol;
  }
}
```

- 위의 BasicOperation은 기본 연산이고, 확장 연산을 제공하려면 아래와 같이 또 다른 Operation을 구현하면 된다.

```java
public enum ExtendedOperation implements Operation {
  EXP("^") {
    public double apply(double x, double y) {
      return Math.pow(x, y);
    }
  }, 
	REMAINDER("%") {
    public double apply(double x, double y) {
      return x % y;
    }
  };
  private final String symbol;

  ExtendedOperation(String symbol) {
    this.symbol = symbol;
  }

  @Override
  public String toString() {
    return symbol;
  }
}
```

- 위와 같이 하면 Operation을 사용하도록 코드를 구현해두고, Operation 변수에 주입시켜줄 때만 구분해주면 된다.

### Client에서 사용하는 방법

1. Class 리터럴(한정적 타입 토큰 역할)을 넘겨서 사용
    - <**T extends Enum<T> & Operation>** 의미: Class 객체가 열거 타입인 동시에 Operation의 하위 타입이어야 한다는 뜻.

    ```java
    public static void main(String[] args) {
      double x = Double.parseDouble(args[0]);
      double y = Double.parseDouble(args[1]);
      test(ExtendedOperation.class, x, y);
    }

    private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x,
        double y) {
      for (Operation op : opEnumType.getEnumConstants()) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
      }
    }
    ```

2. 한정적 와일드카드 타입 사용
    - 위와 비교해서 더 유연해진 구조가 되었다. 왜냐하면 Arrays.asList등을 통해서 여러 Operation을 넣을 수 있기 때문이다.

    ```java
    public static void main(String[] args) {
      double x = Double.parseDouble(args[0]);
      double y = Double.parseDouble(args[1]);
      test(Arrays.asList(ExtendedOperation.values()), x, y);
    }

    private static void test(Collection<? extends Operation> opSet, double x, double y) {
      for (Operation op : opSet) {
        System.out.printf("%f %s %f = %f%n", x,op,y,op.apply(x,y));
      }
    }
    ```

## Enum Interface의 사소한 문제 하나

- 열거 타입끼리 구현을 상속할 수 없다.
    - 이것은 default 메소드나 별도의 도우미 클래스, 정적 도우미 클래스를 만들어서 해결할 수 있을 것이다.

### 추가 Enum 관련 자료

- [Attaching Values to Java Enum](https://www.baeldung.com/java-enum-values)