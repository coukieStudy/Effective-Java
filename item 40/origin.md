## @Override 애너테이션을 일관되게 사용하라

```java
public class Bigram {
  private final char first;
  private final char second;
  
  public Bigram(char first, char second) {
    this.first = first;
    this.second = second;
  }
  
  public boolean equals(Bigram b) {
    return b.first == first && b.second == second;
  }
  
  public int hashCode() {
    return 31 * first + second;
  }

  public static void main(String[] args) {
    Set<Bigram> s = new HashSet<>();
    for (int i = 0; i < 10; i++)
      for (char ch = 'a'; ch <= 'z'; ch++)
        s.add(new Bigram(ch, ch));
        
    System.out.println(s.size());
  }
}
```

Set은 중복된 원소를 허용하지 않기 때문에 위의 코드를 실행하면 26이 출력되어야 하지만 260이 출력된다.


그 이유는 Object.eqauals는 parameter로 Object를 받아야 하는데, Biggram내의 equals는 Biggram을 받았기 때문에 method overloading 을 한 셈이기 때문이다.

```java
@Override public boolean equals(Object o) {
  if (!(o instanceof Bigram))
    return false;
  Bigram b = (Bigram) o;
  return b.first == first && b.second == second;
}
```

***

1. 이를 방지하기 위해 equals앞에 @Override 애노테이션을 붙이면 컴파일 오류가 발생하고 (method overridding이 제대로 이루어지지 않았기 때문에) 오류를 컴파일 시점에 해결할 수 있다. 따라서 상위 클래스의 메서드를 재정의 하는 경우에는 
@Override를 일관적으로 붙이는 것이 좋다.


2. 구체클래스에서 상위 추상 클래스의 메서드를 재정의할 때에는 @Override를 달지 않더라도 컴파일러 시점에 재정의 하지 않은 메서드들을 알 수 있기 때문에 붙이지 않아도 된다.


3. 추상클래스나 인터페이스에서 상위 추상클래스나 인터페이스의 메서드들을 재정의하는 경우에는 재정의하려는 모든 메서드에 @Override를 다는 것이 좋다. 위의 오류(재정의하려고 했으나 실제로는 재정의가 되지 않는 오류)를 찾아내어 실수로 메서드가 추가되는 일을 방지해준다.
