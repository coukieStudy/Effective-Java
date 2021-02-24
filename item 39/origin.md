# Chapter 39 명명 패턴보다 애너테이션을 사용하라.

- 명명 패턴
    - 명명 패턴이란 이름에 근거해서 동작하는 것을 뜻하는데, JUnit3의 경우 test를 prefix로 붙여야 test가 동작했다.
    - 끔찍

## Annotation

- @Test Annotation: 자동으로 수행되는 간단한 Annotation, 예외가 발생하면 해당 테스트를 실패로 처리한다.
    - 아래처럼 Annotation 선언에 다는 Annotation을 **Meta annotation**이라고 한다.
    - @Retention: @Test가 런타임에도 유지되어야 한다는 표시
    - @Target(ElementType.METHOD): @Test가 반드시 메서드 선언에서만 사용돼야 한다고 알려준다.

```java
import java.lang.annotation.*;
/**
* 테스트 메서드임을 선언하는 애너테이션이다. 
* 매개변수 없는 정적 메서드 전용이다.
*/
@Retention(RetentionPolicy.RUNTIME) 
@Target(ElementType.METHOD)
public @interface Test {
}
```

### Marker Annotation

- "아무 매개변수 없이 단순히 대상에 마킹한다"는 뜻에서 마커 Annotation
- ex) @Test, @Override

### @Test Runner

- 실제 @Test Annotation 붙은 것들을 running하기 위해서 만든 것.

```java
public class RunTests {
    public static void main(String[] args) {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]); //Class명을 받아서
				/* 그 안에 Method 중 Test annotation이 붙은 것들을 실행 */
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                test++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception e) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        
        System.out.printf("성공: %d, 실패: %d%n", passed, tests-passed);
    }
}
```

### 예외를 여러 개 명시하고 하나가 발생하면 성공하게 하는 법

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest{
    Class<? extends Throwable>[] values(); //배열로 안 받으면 한가지 예외만 가능
}
```

```java
@ExceptionTest({ IndexOutOfBoundsException.class, NullPointerException.class })
public static void doublyBad() { 
	List<String> list = new ArrayList<>();
	list.addAll(5, null);
}
```

- @ExceptionTest Runner

```java
public class RunTests {

  public static void main(String[] args) {
    int tests = 0;
    int passed = 0;
    Class<?> testClass = Class.forName(args[0]); //Class명을 받아서
    /* 그 안에 Method 중 Test annotation이 붙은 것들을 실행 */
    for (Method m : testClass.getDeclaredMethods()) {
      if (m.isAnnotationPresent(ExceptionTest.class)) {
        test++;
        try {
          m.invoke(null);
					//예외를 던지지 않으면 실패
					System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
        } catch (InvocationTargetException wrappedExc) {
          //exec: 실행 중에 던진 Exception
          Throwable exec = wrappedExc.getCause();
          //excType: 지정한 Exception
          Class<? extends Throwable>[] excTypes = m.getAnnotation(ExceptionTest.class).value();
          int oldPassed = passed;
          for(Class<? extends Throwable> excType : excTypes) {
            if(excType.isInstance(exc)) {
              passed++;
              break;
            }
          }
          if(passed == oldPassed) {
            System.out.println("테스트 %s 실패: %s %n", m, exc);
          }
        } catch (Exception e) {
          System.out.println("잘못 사용한 @Test: " + m);
        }
      }
    }
    System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
  }
}
```

### Java 8에서 사용할 수 있는 Repeatable Annotation

- 배열 방식으로 여러 개를 예외를 선언하는 것이 아니라 각 예외마다 여러 개의 Annotation을 다는 형식
- 장점: 가독성이 좋아진다는데... 잘 모르겠다..

```java
//@ExceptionTest({ IndexOutOfBoundsException.class, NullPointerException.class })
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void m1() {...}
```

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class) //Container Annotation
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

//여러 개의 Annotation을 달기 위한 Container Annotation
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
```

[https://www.baeldung.com/java-annotations-interview-questions#q6-is-there-a-way-to-limit-the-elements-in-which-an-annotation-can-be-applied](https://www.baeldung.com/java-annotations-interview-questions#q6-is-there-a-way-to-limit-the-elements-in-which-an-annotation-can-be-applied)

- [JUnit](//junit) reflection