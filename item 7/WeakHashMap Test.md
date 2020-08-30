### WeakHashMap - Integer WeakReference

- 코드
```java
import java.util.*;

public class Test {

    public static void main(String[] args) {
        WeakHashMap<Integer, String> weakHashMap = new WeakHashMap<>();

        Integer integer1 = 1000;
        Integer integer2 = 2000;

        weakHashMap.put(integer1, "a");
        weakHashMap.put(integer2, "b");

        integer1 = null;
        System.gc();

        weakHashMap.entrySet().forEach(System.out::println);
    }

}
```

- 결과<br>
![스크린샷 2020-08-30 오후 7 34 57](https://user-images.githubusercontent.com/26040955/91656927-e4858a00-eaf7-11ea-99c7-a1414952933b.png)

### WeakHashMap - Integer StrongReference
- Integer 같은 경우 -127 ~ 128 사이는 미리 선언하여 캐싱하여 사용하기 때문에, Java 프로그램이 돌아가고 있는 동안 참조는 계속된다. 따라서 지역참조변수를 null로 선언하더라도 Integer 클래스에서 참조하고 있기 때문에 gc의 대상이 
아니다.

- 값만 아래와 같이 변경했을 때

```java
Integer integer1 = 125;
Integer integer2 = 128;
```
- 결과는 다음과 같다<br>
![스크린샷 2020-08-30 오후 7 40 36](https://user-images.githubusercontent.com/26040955/91657013-ae94d580-eaf8-11ea-92f1-63d4064bc1f6.png)


### WeakHashMap - String StrongReference
- String literal로 한 경우 String pool에서 관리되기 때문에 지역변수 참조가 해제되더라도 gc의 대상이 아니다.
- 코드
```java
import java.util.*;

public class Test {

    public static void main(String[] args) {
        WeakHashMap<String, String> weakHashMap = new WeakHashMap<>();

        String s1 = "coupang";
        String s2 = "hello";

        weakHashMap.put(s1, "a");
        weakHashMap.put(s2, "b");

        s1 = null;

        System.gc();

        weakHashMap.entrySet().forEach(System.out::println);
    }
}
```
- 결과<br>
![스크린샷 2020-08-30 오후 7 42 24](https://user-images.githubusercontent.com/26040955/91657044-eef45380-eaf8-11ea-9520-497b26badfaf.png)


### HashMap

- HashMap은 hashmap 객체가 gc 대상이 되어야 다 같이 처리되는 듯.
- 코드
```java
import java.util.*;

public class Test {

    public static void main(String[] args) {
        HashMap<Integer, String> hashMap = new HashMap<>();

        Integer integer1 = 1000;
        Integer integer2 = 2000;

        hashMap.put(integer1, "a");
        hashMap.put(integer2, "b");

        integer1 = null;
        System.gc();

        hashMap.entrySet().forEach(System.out::println);
    }
}
```
- 결과<br>
![스크린샷 2020-08-30 오후 7 45 56](https://user-images.githubusercontent.com/26040955/91657102-6de98c00-eaf9-11ea-9602-bbd7aa8cd254.png)

