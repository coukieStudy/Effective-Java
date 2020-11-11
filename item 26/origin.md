# Item 25. 톱레벨 클래스는 한 파일에 하나만!

톱레벨 클래스가 한 파일에 여러 개 있을 경우, 컴파일러에 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라진다.

**예시**

```java
// Main.java
public class Main {
  public static void main(String[] args) {
    System.out.println(Utensil.NAME + Dessert.NAME);
  }
}
```
```java
// Utensil.java 
class Utensil {
  static final String NAME = "pan";
}
class Dessert {
  static final String NAME = "cake";
}
```
```java
// Dessert.java 
class Utensil {
  static final String NAME = "pot";
}
class Dessert {
  static final String NAME = "pie";
}
```
```bash
> javac Main.java Dessert.java // compile error
> javac Main.java Utensil.java // print pancake
> javac Dessert.java Main.java // print potpie
```

**해결책**
1. 톱레벨 클래스를 분리
2. 정적 멤버 클래스(item 24) : 위의 예시에서 Utensil, Dessert를 main의 static member class로.
