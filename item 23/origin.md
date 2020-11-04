## 태그달린 클래스보다는 클래스 계층구조를 이용하라

* 태그 달린 클래스란? 두 가지 이상의 의미를 표현할 수 있으며 그 중 현재 표현하는 의미를 태그값으로 알려주는 클래스

```java
//태그 달린 클래스

class Figure {
    enum Shape { RECTANGLE, CIRCLE };
    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

위 코드는 단점이 매우 많다.
1. 우선 열거 타입 선언, 태그 필드, switch 문 등 쓸데없는 코드가 많다. 가독성도 좋지 않을 뿐더러 다른 의미를 위한 코드도 섞여 있어 메모리 사용도 높다.
2. 필드들(위 코드에서 length, width, radius)를 final 로 선언하려면 생성자에서 해당 의미에 쓰이지 않는 필드들까지 초기화해야 한다. 잘못된 초기화에 따른 오류를 찾아내기도 어렵다.
3. 의미를 추가하려면 기존 코드를 수정해야 한다. 위에서 Shape 가 원이나 사각형 이외의 다른 모양을 표현하려고 한다면 switch가 존재하는 모든 함수들을 수정해야 한다.
4. 인스턴스 변수의 타입만으로는 현재 어떤 의미를 나타내고 있는지 알 수가 없다.




따라서 다음과 같은 방법으로 수정한다.

1. 계층구조의 root가 될 추상클래스를 정의하고 하위 의미에 따라 동작이 달라지는 메서드(예컨데 switch가 포함된 메서드)를 추상 메서드로 선언한다.
2. 하위 의미에 관계없이 동작이 일정한 메서드들을 일반 메서드로 추가하고, 모든 하위 의미에서 사용하는 필드들을 루트에 추가한다.
3. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의하고 구현한다.

```java
// root 추상 클래스
abstract class Figure {
    abstract double area();
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    @Override double area() { return length * width; }
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override double area() { return Math.PI * (radius * radius); }
}

class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```
