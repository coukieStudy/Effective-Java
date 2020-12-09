## 익명클래스 보다는 람다를 사용하라

익명 클래스의 인스턴스를 함수 객체로 사용하는 방법.
```java
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});
```

Java 8 의 도입과 함께 추상메서드 1개짜리 인터페이스는 람다식을 이용하여 해당 인터페이스의 객체를 생성할 수 있게 되었다.

람다는 함수나 익명 클래스와 개념은 비슷하지만 코드는 훨씬 간결하다. 자질구레한 코드들이 사라지고 어떤 동작을 하는지가 명확하게 드러난다.

```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

람다식에는 타입을 명시적으로 입력하지 않아도 컴파일러가 알아서 추론해준다. 타입을 명시하는 게 더 명확할 경우, 그리고 컴파일러가 타입 추론을 하지 못하는 경우를 제외하고는 람다식에서 타입을 명시하지 않는 것이 좋다. (코드의 간결성 때문에)
메서드 참조를 이용하면 더 짧게 작성할 수도 있다.

```java
Collections.sort(words, comparingInt(String::length));

//Java8에 추가된 List.sort()
words.sort(comparingInt(String::length));
```

람다를 활용하면 enum 타입에 인스턴스 필드를 이용하는 방식으로 상수별로 다르게 동작하는 코드를 구현할 수 있다.

```java
// 상수별 클래스 몸체를 구현하는 방식
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }

    @Override public String toString() { return symbol; }

    public abstract double apply(double x, double y);
}


//인스턴스 필드를 이용하는 방식
public enum Operation {
    PLUS  ("+", (x, y) -> x + y),
    MINUS ("-", (x, y) -> x - y),
    TIMES ("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override public String toString() { return symbol; }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```


람다는 이름도 없고, 문서화도 하지 못하기 때문에, 코드 자체로 명확히 설명이 되지 않거나 코드의 줄 수가 길어지면 사용하지 말아야 한다.

람다를 쓸 수 없는 경우도 존재한다.
- 추상 클래스의 인스턴스를 만들 경우.(람다는 함수형 인터페이스에서만 쓸 수 있음)
- 인터페이스의 추상메서드가 여러개인 경우.
- 람다는 자신을 참조할 수 없기 때문에, 함수 객체가 자기 자신을 반드시 참조해야 한다면 익명 클래스를 써야 한다.

또한 람다나 익명클래스는 직렬화 형태가 구현별로(가상머신별로) 다를 수 있다. 따라서 직렬화해야 하는 함수 객체가 있다면 private 정적 중첩클래스를의 인스턴스를 활용해야 한다. ....??

