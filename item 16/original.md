# public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라


클래스를 작성하는 이유가 없어지는 클래스
예시) 
```java
class public{
  public double x;
  public double y;
}
```

접근자와 변경자(mutator) 메서드를 활용해 데이터를 캡슐화한다.
```java
class Point {
  private double x;
  private double y;
  
  public Point(double x, double y) {
    this.x = x;
    this.y = y;
  }
  
  public double getX() { return x; }
  public double getY() { return y; }
  
  public void setX(double x) { this.x = x; }
  public void setY(double y) { this.y = y; }
}
```
* 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제드 바꿀 수 있는 유연성을 얻을 수 있다.

public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다. 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. 하지만 package-private 클래스나 private 중 첩 클래스에서는 종종 (불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.

