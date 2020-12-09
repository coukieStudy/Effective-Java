# 표준 함수형 인터페이스를 사용하라
- 자바가 람다를 지원하면서 좋은 API를 작성하는 방법도 바뀌었다.
- 예전에는 템플릿 메서드 패턴을 사용해 작성하던 것을, 이제는 함수 객체를 인자로 받는 정적 팩터리나 생성자를 제공하는 것이 더 좋은 방법이다.

## 예시: `LinkedHashMap`
- 현재 `LinkedHashMap`은 `put` 메서드를 호출하면 내부에서 `removeEldestEntry`를 호출해 그 값이 `true`이면 맵에서 가장 오래된 원소를 제거한다. 
- 여기서 `removeEldestEntry` 메서드를 재정의하면 맵을 캐시로 이용할 수 있다. 이것이 원래 개발자가 의도한 방식 (템플릿 메서드 패턴)이다.
- 이 방식을 이용하면 다음과 같이 구현할 수 있다.
```java
@Override
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return size() > 100;
}
```
- 이렇게 하면 맵은 가장 최근 원소 100개만을 유지한다.
- 같은 로직을 람다로 구현한다면? 팩터리나 생성자에 `removeEldestEntry`와 똑같은 동작을 하는 함수를 인자로 넣을 수 있도록 만들면 된다.
- 그 함수에는 무엇이 인자로 들어가야 할까? `removeEldestEntry`에서는 `size()` 메서드를 썼지만 이건 우리 함수에서 쓸 수 없다. 인스턴스가 없기 때문이다. 따라서 맵 인스턴스를 따로 인자로 받아야 한다.
```java
@FunctionalInterface interface EldestEntryRemovalFunction<K,V> {
    boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}
```
- 이 함수는 인자 두 개를 받아 `boolean` 타입을 리턴한다.
- 그러데 사실 이런 인터페이스를 따로 만들 필요는 없다. 자바에서 정확히 같은 인터페이스를 기본으로 제공해주기 때문이다. 그것이 바로 `BiPredicate`이다.
- `BiPredicate`로 현대적인 `LinkedHashMap`을 만들어보자.
```java
public class LinkedHashMap<K,V>
extends HashMap<K,V> implements Map<K,V> {
    BiPredicate<Map<K,V>,Map.Entry<K,V>> eldestEntryRemovalFunction;

    public LinkedHashMap(BiPredicate<Map<K,V>,Map.Entry<K,V>> eldestEntryRemovalFunction) {
        this.eldestEntryRemovalFunction = eldestEntryRemovalFunction;
    }

    void afterNodeInsertion(boolean evict) {
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && eldestEntryRemovalFunction != null
        && eldestEntryRemovalFunction.test(this, first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }
}
```
- 그러면 가장 최근 원소 100개만 유지하는 맵은 다음과 같이 생성할 수 있다.
```java
Map<K,V> map = new LinkedHashMap<>((map, entry) -> map.size() > 100);
```

## 표준 함수형 인터페이스
- `BiPredicate`와 같이 자바에서는 여러 종류의 함수형 인터페이스를 미리 만들어 제공하고 있다. 이를 **표준 함수형 인터페이스**라고 한다.
- 용도에 맞는 게 있다면 따로 만들지 말고 표준 함수형 인터페이스를 사용하라.
### 기본 인터페이스
- `UnaryOperator<T>`: 리턴값과 인자의 타입이 같은 함수 (인자 1개)
- `BinaryOperator<T>`: `UnaryOperator`와 같으나 인자가 2개
- `Predicate<T>`: 인자 하나를 받아 `boolean` 타입을 리턴하는 함수
- `Function<T,R>`: 리턴값과 인자의 타입이 다른 함수 (인자 1개)
- `Supplier<T>`: 인자는 없고 리턴값만 제공하는 함수
- `Consumer<T>`: 인자 하나를 받고 리턴값은 없는 (void) 함수
### 응용 인터페이스
- 각 기본 인터페이스는 3개의 기본 타입(`int`, `long`, `double`)을 인자로 받기 위해 3개씩 변형이 존재한다. (e.g. `IntPredicate`, `LongPredicate`, `DoublePredicate`)
- `Function`은 리턴값과 인자의 타입이 다르므로 기본 타입을 위한 변형에도 매개변수 타입이 있다. (e.g. `LongFunction<int[]>`)
- `Function`은 3개의 기본 타입을 리턴값으로 쓰기 위해 9개의 변형이 더 존재한다. (e.g. `LongToIntFunction`, `ToLongFunction<int[]>`)
- `Predicate`, `Function`, `Consumer`는 인자를 2개 받는 변형이 존재한다. (e.g. `BiPredicate<T,U>`, `BiFunction<T,U,R>`, `BiConsumer<T,U>`)
- `BiFunction`은 3개의 기본 타입을 리턴값으로 쓰기 위해 3개의 변형이 더 존재한다. (e.g. `ToIntBiFunction<T,U>`, ...)
- `Consumer`는 참조 타입과 기본 타입 하나씩 인자를 받는 3개의 변형이 더 존재한다. (e.g. `ObjDoubleConsumer<T>`, ...)
- `Supplier`는 `boolean` 값을 반환하기 위한 변형인 `BooleanSupplier`가 존재한다.
- 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하자 마라. 느려진다.

## 언제 인터페이스를 직접 작성해야 하는가?
- 표준 인터페이스에 원하는 게 없을 때
- 표준 인터페이스와 구조적으로 같더라도, 직접 작성하는 게 더 유용할 때: `Comparator`
    - 자주 쓰이며, 이름 자체가 self-explanatory하다.
    - 반드시 따라야 하는 규약이 있다. (크면 리턴값이 양수, 같으면 0, 작으면 음수)
    - 유용한 디폴드 메서드와 정적 메서드를 제공한다.

## `@FunctionalInterface` 애너테이션
- 함수형 인터페이스임을 알려주는 애너테이션
- 두 가지 목적이 있다.
    - 가독성
    - 해당 인터페이스가 추상 메서드를 하나만 가지고 있어야 컴파일되도록 함

## 오버로딩시 주의점
- 오버로딩할 때 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 만들면 안 된다.
- 인터페이스 여러 개를 구현한 인터페이스가 존재할 수 있기 때문이다.
- 예시: `ExecutorService`
```java
public interface ExecutorService extends Executor {
    <T> Future<T> submit(Callable<T> task);

    Future<?> submit(Runnable task);
}
```
- 이렇게 구현된 메서드에 다음과 같은 인터페이스가 인자로 들어올 수 있다.
```java
final class SchedulerTask implements Runnable, Disposable, Callable<Void> {}
```
- 그러면 이렇게 형변환해줘야 쓸 수 있다.
```java
exec.submit((Callable<?>) sr);
```

## 번외: 다른 언어에서는 어떻게 할까?
### 파이썬 (>= 3.5)
- `Callable[[Arg1Type, Arg2Type], ReturnType]`을 이용해 구현한다.
```python
from collections.abc import Callable

def feeder(get_next_item: Callable[[], str]) -> None:
    # Body

def async_query(on_success: Callable[[int], None],
                on_error: Callable[[int, Exception], None]) -> None:
    # Body
```
### 타입스크립트
- 자바처럼 인터페이스가 존재한다. 다만 함수형 인터페이스의 경우 메서드를 따로 정의하지 않아도 된다.
- 표준 함수형 인터페이스는 따로 없는 것 같다.
```typescript
interface numberOperation {
  (arg1: number, arg2: number): number;
}

const sum: numberOperation = (arg1: number, arg2: number): number => {
  return arg1 + arg2;
};

const multiply: numberOperation = (arg1, arg2) => {
  return arg1 * arg2;
};
```
### Go
- 함수 타입을 정의할 수 있다.
```go
package main

import "fmt"

type myFunctionType = func(a, b string) string

func main() {
 var explicit myFunctionType = func(a, b string) string {
  return fmt.Sprintf("%s %s", a, b)
 }

implicit := func(a, b string) string {
  return fmt.Sprintf("%s %s", b, a)
 }

functionTypeConsumer(explicit)
functionTypeConsumer(implicit)
functionTypeConsumer(func(a, b string) string {
 return fmt.Sprintf("%s %s!", a, b)
})}

func functionTypeConsumer(fn myFunctionType) {
 s := fn("hello", "world")
 fmt.Println(s)
}
```
### C# (.NET)
- Delegate라는 것을 사용한다. Delegate란 다음과 같이 메서드의 타입을 정의한 것으로 메서드를 delegate에 위임하여 호출할 수 있다.
```c#
public delegate void Del(string message);

// Create a method for a delegate.
public static void DelegateMethod(string message)
{
    Console.WriteLine(message);
}

// Instantiate the delegate.
Del handler = DelegateMethod;

// Call the delegate.
handler("Hello World");
```
- .NET은 자바의 표준 함수형 인터페이스 비스무리하게 기본 delegate 두 개를 제공하는데 바로 `Action`과 `Func`이다. `Action`은 리턴값이 없는 delegate이고 `Func`는 리턴값이 있는 delegate이다.
- C#에서는 기본 타입도 참조 타입이어서 이 두 개로 충분하다.
- `Action`과 `Func` 모두 인자 개수를 마음대로 쓸 수 있다. .NET 공식 문서의 Delegate 클래스 항목(https://docs.microsoft.com/ko-kr/dotnet/api/system.delegate?redirectedfrom=MSDN&view=net-5.0)을 보면 알 것이다.
```c#
class Program
{
    static int Sum(int x, int y)
    {
        return x + y;
    }

    static void Main(string[] args)
    {
        Func<int,int, int> add = Sum;

        int result = add(10, 10);

        Console.WriteLine(result); 
    }
}
```
- 결론: 파이썬 > C# >>>> Go >= TS >= 자바