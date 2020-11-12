# Item 27. 비검사 경고(unchecked warning)를 제거하라

## 제네릭을 사용하기 시작하면 수많은 컴파일러 경고를 보게 될 것이다.
비검사 형변환 경고, 비검사 메서드 호출 경고, 비검사 매개변수화 가변인수 타입 경고, 비검사 변환 경고 등

**할 수 있 는 한 모든 비검사 경고를 제거하라.**

모두 제거한다면 그 코드는 타입 안전성이 보장된다. 즉, 런타임에 ClassCastException이 발생할 일이 없다.

**경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 @Suppress Warnings("unchecked") 애너테이션을 달아 경고를 숨기자.**
@SuppressWarnings 애너테이션은 항상 가능 한 한 좁은 범위에 적용하자.

=> 경고를 주의 깊게 살펴서 코드를 작성하자

ex) toArray 코드 27-1 지역변수를 추가해 @SuppressWarnings의 범위를 좁힌다.
```java 
public <T> T[] toArray(T[] a) {
  if (a.length < size) {
  // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로 
  // 올바른 형변환이다.
    @SuppressWarnings("unchecked") T[] result = 
      (T[]) Arrays.copyOf(elements, size, a.getClass());
    return result;
  }
  System.arraycopy(elements, 0, a, 0, size);
  if (a.length > size)
    a[size] = null;
  return a;
}
```
**@SuppressWarnings("unchecked") 애너테이션을 사용할 때면 그 경고를 무시 해도 안전한 이유를 항상 주석으로 남겨야 한다.**
