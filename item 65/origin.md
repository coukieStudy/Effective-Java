# 리플렉션보다는 인터페이스를 사용하라.

### 리플렉션(java.lang.reflect)의 기능

1. class 객체가 주어지면 그 클래스의 Constructor, Method, Field 인스턴스를 가져올 수 있다.
2. 그 인스턴스를 이용해 생성자, 메서드, 필드를 조작할 수 있다. 
3. 컴파일 당시에 존재하지 않던 클래스도 이용할 수 있다.

### 리플렉션의 단점

1. 컴파일타임 타입 검사의 이점을 누릴 수 없다.
2. 코드가 지저분하고 성능이 떨어진다.

따라서 리플렉션은 제한된 형태로만 사용해야 그 단점을 피하고 이점을 취할 수 있다.<br>

=> **리플렉션은 인스턴스 생성에만 쓰고, 이렇게 만든 인스턴스는 인터페이스나 상위 클래스를 참조해서 사용하자.**

### 예제

```java
public static void main(String[] args) {
    
    // 클래스 이름을 Class 객체로 변환
    Class<? extends Set<String>> cl = null;
    try {
        cl = (Class<? extends Set<String>>) Class.forName(args[0]); // 비검사 형변환 warning
    } catch (ClassNotFoundException e) {
        fatalError("클래스를 찾을 수 없습니다.");
    }
    
    // 생성자를 얻는다.
    Constructor<? extends Set<String>> cons = null;
    try {
        cons = cl.getDeclaredConstructor();
    } catch (NoSuchMethodException e) {
        fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
    }
     
    //집합의 인스턴스를 만든다.
    Set<String> s = null;
    try {
        s = cons.newInstance();
    } catch (IllegalAccessException e) {
        fatalError("생성자에 접근할 수 없습니다.");
    } catch (InstantiationException e) {
        fatalError("클래스를 인스턴스화할 수 없습니다.");
    } catch (InvocationTargetException e) {
        fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
    } catch (ClassCastException e) {
        fatalError("Set을 구현하지 않은 클래스입니다.");
    }
    
    //생성한 집합을 사용한다.
    s.addAll(Arrays.asList(args).subList(1, args.length));
    System.out.println(s);
}
```

위의 예제에서 리플렉션은 생성에만 사용하고 인터페이스를 활용한다. <br>

**위 코드의 단점**

- 런타임에 예외 발생 가능.
- 클래스 이름만으로 인스턴스를 생성하기 위해 코드가 지저분해짐.

=> 모두 객체를 생성하는 부분에서 발생하는 단점이다.
