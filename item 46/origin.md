# 스트림에서는 부작용 없는 함수를 사용하라
- 스트림은 함수형 프로그래밍에 기초한 패러다임이다. 진정으로 스트림의 이점을 얻으려면 API뿐만 아니라 이 패러다임까지 함께 받아들여야 한다.
- 스트림 패러다임의 핵심은 계산을 일련의 변환(transformation)으로 재구성하는 부분이다. 여기서 각 변환 단계는 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다.
    - 순수 함수: 입력만이 결과에 영향을 미치는 함수. 다른 가변 상태를 참조하지 않고 함수 자체도 다른 상태를 변경하면 안된다.
    - 즉 스트림 연산에 사용되는 함수 객체는 부작용이 없어야 한다.
- 나쁜 예시: 빈도표
```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    // 외부의 freq를 변경하고 있다
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```
- 위 코드는 스트림을 가장한 반복문이다. 오히려 반복문보다 읽기 어렵다.
- `forEach`를 for-each 반복문처럼 사용하면 안 된다. 오직 스트림 계산 결과를 보고할 때만 사용해야 한다.
- 올바르게 스트림을 사용한 예시
```java
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words
            .collect(groupingBy(String::toLowerCase, counting()));
}
```
## 수집기
- 수집기는 스트림의 원소들을 컬렉션으로 모으는 종단 연산이다.
- 수집기는 `java.util.stream.Collectors` 클래스에 총 39개가 존재한다.
- 기본 3개 수집기: `toList()`, `toSet()`, `toCollection(collectionFactory)`
```java
// 빈도표에서 가장 흔한 단어 10개를 리스트로 뽑아내는 스트림 파이프라인
List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reversed())
    .limit(10)
    .collect(toList());
```
### `toMap`
- `toMap(keyMapper, valueMapper)`: 스트림을 맵에 매핑하는 가장 간단한 수집기, 모든 원소가 다른 키에 매핑될 때만 사용할 것
```java
private static final Map<String, Operation> stringToEnum = 
    Stream.of(values()).collect(toMap(Object::toString, e -> e));
```
- `toMap(keyMapper, valueMapper, mergeFunction)`: 같은 키를 공유하는 원소들을 맵으로 수집할 수 있다.
    - 각 키와 해당 키의 특정 원소를 연관짓는 맵을 생성하는 예시
    ```java
    Map<Artist, Album> topHits = albums.collect(
        toMap(Album::artist, a -> a, maxBy(comparing(Album::sales)))
    );
    ```
    - 마지막에 쓴 값을 취하는 예시
    ```java
    toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
    ```
- `toMap(keyMapper, valueMapper, mergeFunction, mapSupplier)`: 특정 맵 구현체를 지정할 수 있다.
- `toConcurentMap`: `ConcurrentHashMap`을 리턴한다.
### `groupingBy`
- `groupingBy`: 분류 함수를 인자로 받아 맵을 만드는 수집기를 반환한다. 맵의 값은 리스트가 된다.
    - 아나그램 수집기 예시
    ```java
    words.collect(groupingBy(word -> alphabetize(word)))
    ```
- 다운스트림 수집기를 두 번째 인자로 주면 리스트 외에 다른 타입을 값으로 받을 수 있다.
    - `toSet()`을 넘기는 경우
    ```java
    words.collect(groupingBy(word -> alphabetize(word), toSet()))
    ```
    - 빈도표 만들기
    ```java
    words.collect(groupingBy(String::toLowerCase, counting()))
    ```
- `groupingBy(classifier, mapFactory, downStream)`: `mapFactory`로 원하는 맵 팩터리를 지정할 수 있다.
- `groupingByConcurrent`: `ConcurrentHashMap`을 만들어준다.
- `partitioningBy(predicate)`: 키가 boolean인 맵을 리턴한다.
- `counting`, `summing`, `averaging`, `summarizing` 등등: 몰라도 된다.
### `minBy`, `maxBy`, `joining`
- `Collectors`에 정의되어 있지만 수집과는 관련이 없는 메서드들이다.
- `minBy`, `maxBy`: 각각 스트림에서 min, max 값을 찾아 리턴한다.
- `joining`: `charSequence` 스트림에만 사용할 수 있는 수집기로 문자들을 연결해 문자열을 만드는 수집기를 리턴한다. delimiter, prefix, suffix를 인자로 받을 수 있다.