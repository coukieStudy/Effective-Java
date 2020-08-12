## 생성자가 매개변수가 많다면 빌더를 고려하라

### 점층적 생성자 패턴(telescoping constructor pattern)

- 필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 선택 매개변수를 2개..3개
  * 아래의 경우 인스턴스를 만드려고 할 때 원하는 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 골라 호출하면 된다.
  * 원하지 않는 매개변수에도 초기화 값 전달 vs 생성자가 너무 많아짐

```java
public NutritionFacts(int servingSize, int servings){

}
public NutritionFacts(int servingSize, int servings, int calories){

}
public NutritionFacts(int servingSize, int servings, int calories, int fat){

}
```

- 따라서 점층적 패턴도 쓸 수 있지만, 매개변수가 많아지면 코드를 작성하거나 읽기 어렵다.
  * 만약 클라이언트가 실수로 매개변수의 순서를 바꿔 건네주게 되면, 런타임에 엉뚱한 동작을 하게 된다.(Item 51)
  * 함수에 매개변수 4개 이하, 같은 종류의 타입이 연속 해롭다. 실수 가능성 높아짐.

### 자바 빈즈 패턴(JavaBeans Pattern)
- 매개 변수가 없는 생성자로 객체를 만든 후, Setter 메서드를 호출해 원하는 매개변수의 값을 설정하는 방식.
  * setter로 만들면 더이상 점층적 생성자 패턴의 단점(코드 작성/가독성 X)들이 보이지 않는다.

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```
- 하지만 치명적인 단점: 불변으로 만들 수 없다.
  * 점층적 생성자 패턴에서는 매개변수들이 유효한지를 생성자에서만 확인하면 일관성을 유지할 수 있는데, setter를 사용하면 어디서든지 변경될 여지가 있다.
  * 자바빈즈 패턴에서 불변으로 만들 수 없으며 스레드 안전성을 얻으려면 프로그래머가 추가 작업을 해줘야만 한다.

- 따라서 런타임 오류에 취약하다.
  
### 자바 빈즈 패턴이 Thread-safe하지 않은 이유 && 해결책
- Thread-safe하지 않는 이유
  * 다른 thread가 현재 thread가 초기화하기 전의 값을 볼 수 있다.
  * 다른 thread가 instance를 볼 때, 올바른 값을 볼 수 없다.
- 해결책
  * synchronized 키워드: 상관없음(가시성 보장)
  * volatile 키워드: 하나의 스레드만 쓰고, 나머지 스레드가 읽기일 경우 (가시성 보장)
  
- 참고
  * [Why Java Bean Pattern is not threadsafe](https://stackoverflow.com/questions/44345780/whyjava-bean-pattern-is-not-threadsafe)
  * [자바 volatile 키워드](https://parkcheolu.tistory.com/16)


### Builder Pattern
- 점층적 생성자 패턴의 안전성과 자바 빈즈 패턴의 가독성을 겸비한 것.
- 과정
  1. 필수 매개변수만으로 생성자(or 정적 팩토리)를 호출해 빌더 객체를 얻는다. 
  2. 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정.
  3. 매개변수가 없는 build 메서드를 호출해 (보통은 불변인)
- 불변인 예시
```java
private final int servingSize;
private final int servings;
private final int calories;
private final int fat;
```


- 아래와 같이 builder가 메서드 호출이 흐르듯 연결되는 것을 Fluent API 혹은 Method Chaining이라고 한다.
```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
  .calories(100).sodium(35).carbohydrate(27).build();
```

- 위에서 말했듯이 매개변수에 잘못된 값을 넣게 되면 런타임에 문제가 발생할 수 있는데, 빌더의 생성자와 메서드에서 입력 매개변수를 검사하고, build 메서드가 호출하는 생성자에서 여러 매개변수에 걸친 불변식을 검사하자.
  * 참고: https://madplay.github.io/post/make-defensive-copies-when-needed
```java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());

    if(start.compareTo(end) > 0) {
        //검사해서 잘못되었기에 exception
        throw new IllegalArgumentException(start + " after " + end);
    }
}
```

### 불변(Immutable) vs 불변식(Invariant)
- 불변: 어떠한 변경도 허용하지 않는다는 뜻. 대표적으로 Wrapper Class, String은 Immutable하다.
- 불변식: 프로그램이 실행되는 동안, 혹은 정해진 기간 동안 반드시 만족해야 하는 조건. 다시 말해 변경을 허용할 수는 있지만, 조건을 만족해야 한다.
  * 위와 같이 start가 end보다 크면 안되는 예시
  * 따라서 가변 객체에도 불변식은 존재 가능. 


### 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다

- 각 계층의 클래스에 관련 빌더를 멤버로 정의하자. 추상 클래스는 추상 빌더를 구체 클래스는 구체 빌더를 갖게 한다.
```java
public abstract class Pizza {
  public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE } final Set<Topping> toppings;
  abstract static class Builder<T extends Builder<T>> { 
    EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class); 
    public T addTopping(Topping topping) {
      toppings.add(Objects.requireNonNull(topping));
      return self();
    }
    abstract Pizza build();
    // 하위 클래스는 이 메서드를 재정의(overriding)하여 // "this"를 반환하도록 해야 한다.
    protected abstract T self();
  }
  Pizza(Builder<?> builder) {
    toppings = builder.toppings.clone(); // 아이템 50 참조 }
  }
}
```

```java
//뉴욕 피자
public class NyPizza extends Pizza {
  public enum Size { SMALL, MEDIUM, LARGE } 
  private final Size size;
  public static class Builder extends Pizza.Builder<Builder> {
    private final Size size;
    public Builder(Size size) {
      this.size = Objects.requireNonNull(size); 
    }
    @Override public NyPizza build() {
      return new NyPizza(this); 
    }
    @Override protected Builder self() { 
      return this; 
    } 
  }
  private NyPizza(Builder builder) { 
    super(builder);
    size = builder.size; 
  }
}

//똑같은 방식으로 칼초네 피자도 존재
```

- 계층적 빌더를 사용하더라도 객체를 생성하는 것은 일반 빌더와 다르지 않다. 하지만 장점은 가변인수 매개변수를 여러 개 사용할 수 있다. 여기서 '토핑'이 이것에 해당한다.
  * 생성자로서는 누릴 수 없는 이점이다.
- 빌더 패턴은 상당히 유연하다. 빌더 하나로 여러 객체를 순회하면서 만들 수 있고, 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수도 있다. 객체마다 부여되는 일련번호와 같은 특정 필드는 빌더가 알아서 만들도록 할 수 있따.
  * 일련번호 같은 경우 Builder에 함수를 만들지 않으면 된다.
- Pizza.Builder 클래스는 재귀적 타입 한정(Item 30) TODO)

### Builder 패턴의 단점
- 빌더 생성 비용이 크지는 않지만 성능에 민감한 상황에서는 문제가 될 수 있다..(그런 use-case가 있나..)
- 또한 점층적 생성자 패턴보다는 코드가 장황해서 매개변수가 4개 이상은 되어야 값어치를 한다.
  * 하지만 API는 시간이 지날수록 매개변수가 많아지는 경향이 있음을 명심.

### 결론
- 상황을 고려해서 매개변수가 적을 때는 생성자나 정적 팩터리 방식으로 시작했다가 나중에 매개변수가 많아지면 빌더 패턴으로 전환할 수 있지만, 애초에 빌더로 시작하는 편이 나은 경우가 많다...




