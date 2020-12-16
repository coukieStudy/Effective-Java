# Item 52. 다중정의는 신중히 사용하라

###### 코드 52-1 컬렉션 분류기 - 오류! 이 프로그램은 무엇을 출력할까?

```java
public class CollectionClassifier {
  public static String classify(Set<?> s) {
    return "집합";
  }

  public static String classify(List<?> lst) {
    return "리스트";
  }

  public static String classify(Collection<?> c) {
    return "그 외";
  }

  public static void main(String[] args) {
    Collection<?>[] collections = {
      new HashSet<String>(),
      new ArrayList<BigInteger>(),
      new HashMap<String, String>().values()
    };
    for (Collection<?> c : collections)
      System.out.println(classify(c));
  }
}
```

결과 : “그 외” 만 세 번 연달아 출력한다.

**왜?** 다중정의(overloading, 오버로딩) 된 세 classify 중 **어느 메서드를 호출할지가 컴파일타임에 정해지기 때문**이다.

이처럼 직관과 어긋나는 이유는 **재정의(overriding)한 메서드는 동적으로 선택되고, 다중정의(overloading)한 메서드는 정적으로 선택되기 때문**이다.

###### 해결방법

```java
public static String classify(Collection<?> c) {
  return c instanceof Set ? "집합" :
  			 c instanceof List ? "리스트" : "그 외";
}
```

**안전하고 보수적으로 가려면 매개변수 수가 같은 다중정의는 만들지 말자.**가변인수(varargs)를 사용하는 메서드라면 다중정의를 아예 하지 말아야 한다. 다중정의하는 대신 메서드 이름을 다르게 지어주는 길도 항상 열려 있으니 말이다.

> 예) writeBoolean(boolean), writeInt(int), writeLong(long)



#### 매개변수 수가 같은 다중정의 메서드가 많더라도

매개변수 중 하나 이상이 “근본적으로 다르다(radically different)”면 헷갈릴 일이 없다. 근본적으로 다르다는 건 두 타입의 (null이 아닌) 값을 서로 어느 쪽으로든 형변환할 수 없다는 뜻이다. 

###### 오토박싱도 생각해야한다...

```java
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();
        for (int i = -3; i < 3; i++) {
            set.add(i);
            list.add(i);
        }
        for (int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove(i);
        }
        System.out.println(set + " " + list);
    }
}
```

결과 : ~~"[-3, -2, -1] [-3, -2, -1]"~~   “[-3, -2, -1] [-2, 0, 2]”

왜? list.remove(i)는 다중정의된 remove(int index)를 선택

```java
for (int i = 0; i < 3; i++) {
  set.remove(i);
  list.remove((Integer) i); // 혹은 remove(Integer.valueOf(i)) 
}
```

이 예가 혼란스러웠던 이유는 List<E> 인터페이스가 remove(Object)와 remove(int)를 다중정의했기 때문이다. 제네릭이 도입되기 전인 자바 4까지의 List에 서는 Object와 int가 근본적으로 달라서 문제가 없었다.



###### 람다도 문제를 일으킨다...

```java
// 1번. Thread의 생성자 호출
new Thread(System.out::println).start();
// 2번. ExecutorService의 submit 메서드 호출 
ExecutorService exec = Executors.newCachedThreadPool(); exec.submit(System.out::println);
```

1번과 2번이 모습은 비슷하지만, 2번만 컴파일 오류가 난다.

왜? submit 다중 정의 메서드 중에는 Callable<T>를 받는 메서드도 있기 때문

**다중정의된 메서드(혹은 생성자)들이 함수형 인터페이스를 인수로 받을 때, 비록 서로 다른 함수형 인터페이스라도 인수 위치가 같으면 혼란이 생긴다.**



###### 원칙을 지키지 못한 String 라이브러리의 valueOf

```java
public static String valueOf(char data[]) {
  return new String(data);
}
public static String valueOf(Object obj) {
  return (obj == null) ? "null" : obj.toString();
}
```



