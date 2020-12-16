# Item 48. 스트림 병렬화는 주의해서 사용하라

동시성 프로그래밍을 올바로 작성하려면 안정성(safety)과 응답 가능(liveness) 상태를 유지해야 한다.



```java
public static void main(String[] args) {
  primes()
    //.parallel()
    .map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
    .filter(mersenne -> mersenne.isProbablePrime(50))
    .limit(20)
    .forEach(System.out::println);
}

public static Stream<BigInteger> primes() {
  return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
```

*parallel*을 사용할 경우, 스트림이 파이프라인을 병렬화하는 방법을 찾지 못하고 결과를 출력하지 못한다.

**데이터 소스가 Stream.iterate거나 중간 연산 limit을 쓰면 파이프라인 병렬화로는 성능 개선을 기대하기 어렵다.**

<br>

**스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int 범위, long 범위일 때 병렬화의 효과가 좋다.**

- 이 자료구조들은 모두 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어서 일을 다수의 쓰레드에 분배하기에 좋다는 특징이 있습니다. (`spliterator`)
- 원소들을 순차적으로 실행할 때의 `참조 지역성`(locality of reference, 메모리에 연속해서 저장됨)이 뛰어나다. 참조 지역성이 낮으면 스레드는 데이터가 주메모리에서 캐시메모리로 전송되어 오기를 기다리며 낭비하는 시간이 늘어난다.

<br>

**종단 연산 중 병렬화에 가장 적합한 것은 축소(reduction)이다.** ex) min, max, sum, count, anyMatch, allMatch, noneMatch는 적합하지만 collect은 컬렉션들을 합치는 부담이 크기 때문에 적합하지 않다.

<br>

**스트림을 잘못 병렬화하면 결과가 잘못되거나 오동작(안전 실패, safety failure)할 수도 있다.**

안전 실패는 병렬화한 파이프라인이 사용하는 mappers, filters 혹은 프로그래머가 제공한 다른 함수 객체가 명시한대로 동작하지 않을 때 발생할 수 있다. 따라서 Stream은 이 때 사용되는 함수 객체에 대해 규약을 정의했다.

- Stream의 reduce 연산에 건네지는 accumulator(누적기)와 combiner(결합기) 함수는 반드시 **결합법칙**을 만족해야 한다.
  <br>(결합 법칙 : (a op b) op c = a op (b op c))
- 간섭받지 않아야 한다 (non-interfering) - 파이프라인이 수행되는 동안 **데이터소스가 변경되지 않아야한다.**
- 상태를 갖지 않아야 한다. (stateless) 

위의 요구사항을 지키지 못하더라도 순차적으로 실행하면 올바른 결과를 얻을 수 있다. 하지만 병렬로 수행하면 기대한 결과가 나오지 않을 수 있고, 실패할 수 있으니 주의해야 한다.
<br>


**스트림 병렬화는 오직 성능 최적화 수단이다!**

파이프라인이 수행하는 작업이 병렬화에 드는 추가 비용을 상쇄하지 못한다면 성능 향상은 미미할 수 있다. 조건이 잘 갖춰지면 엄청난 성능 향상을 경험할 수 있다.

아래의 경우에는 병렬화의 이득을 얻을 수 있다.

```java
// n보다 작거나 같은 소수의 개수를 계산.
static long pi(long n) { 
    return LongStream.rangeClosed(2, n)
      // .parallel()
      .mapToObj(BigInteger::valueOf)
      .filter(i -> i.isProbablePrime(50))
      .count();
}
```

