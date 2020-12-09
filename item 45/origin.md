# 스트림은 주의해서 사용하라
- 스트림 API의 핵심 개념 두 가지
    - 스트림: 데이터 원소의 유한/무한 시퀀스
    - 스트림 파이프라인: 스트림으로 수행하는 연산 단계
- 컬렉션, 배열, 정규표현식 패턴 패처 등등에 스트림을 사용할 수 있다.
- 원소 타입은 `int`, `long`, `double` 기본 타입과 참조 타입만 받는다.
- 스트림 연산의 종류
    ```java
    List<String> result = 
    source.stream().map(String::toUpperCase).collect(Collectors.toList());
    //    소스 스트림       중간 연산                종단 연산
    ```
    - 중간 연산
        - 스트림을 다른 스트림으로 변환하는 연산
        - 필터, 맵 등
        - 변환 후 스트림 원소의 타입은 변환 전과 다를 수 있다.
    - 종단 연산
        - 스트림에 마지막 연산을 하여 보통의 자바 타입으로 변환하는 연산
        - 컬렉션에 담기, 원소 하나 선택하기, 출력 등
- 스트림의 특징
    - 지연 평가: 스트림 파이프라인은 종단 연산이 호출되는 시점에 평가된다.
    - 메서드 연쇄가 가능하다 (fluent API)
    - 순차적으로 수행된다.

## 예시: 아나그램
### 반복문 아나그램
```java
// 사전에 있는 단어들을 순화하며 아나그램끼리 묶어서 minGroupSize보다 길이가 긴 것만 출력하는 프로그램
public class IterativeAnagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next();
                // computeIfAbsent는 첫 번째 인자로 주어진 키가 없을 경우
                // 두 번째 인자로 주어진 연산을 수행한다
                groups.computeIfAbsent(alphabetize(word),
                        (unused) -> new TreeSet<>()).add(word);
            }
        }

        for (Set<String> group : groups.values())
            if (group.size() >= minGroupSize)
                System.out.println(group.size() + ": " + group);
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```
### 스트림 아나그램
```java
public class StreamAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                    groupingBy(word -> word.chars().sorted()
                            .collect(StringBuilder::new,
                                    (sb, c) -> sb.append((char) c),
                                    StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }
}
```
- 짧지만 읽기 어렵다.
- 스트림을 남용하면 이런 식으로 읽거나 유지보수하기 어려워진다.
### 절충적 아나그램
```java
public class HybridAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```
- `alphabetize` 메서드를 사용해 리팩터링했다.
- 반복문보다 짧으면서 스트림보다 이해하기 쉽다.
- `alphabetize`도 스트림으로 구현할 수 있지만, `char` 타입에는 스트림을 쓰지 않는 것이 좋다.
    - 일단 자바는 `char`용 스트림을 지원하지 않는다.
    - `char` 타입에 스트림을 쓰는 예시
    ```java
    "Hello world!".chars().forEach(System.out::print);
    // 결과: 7210110810...
    ```
    - 왜 숫자가 출력될까? `chars()` 메서드가 `int` 스트림을 반환하기 때문이다.
    - 형변환을 명시적으로 해주는 예시
    ```java
    "Hello world!".chars().forEach(x -> System.out.print((char) x));
    ```
    이런 식으로 어거지로 가능하더라도 쓰지 않는 게 좋다.
- 스트림과 반복문은 적절히 조합하는 것이 가장 좋다.

## 언제 스트림을 써야 하는가?
- 스트림은 함수 객체를, 반복문은 코드 블록을 사용해 표현한다.
- 코드 블록에서만 가능한 일들 (이런 게 필요하면 반복문을 쓰자)
    - 스코프 안의 지역변수를 읽고 수정하기: 람다에서는 final이거나 effectively final인 변수만 읽을 수 있다.
    - return, break, continue, throw
- 스트림을 쓰기 알맞은 일들
    - 맵
    - 필터
    - 리듀스
    - 컬렉션에 모으기 (그루핑 포함)
    - 특정 조건을 만족하는 원소 찾기 (min, max 등)
- 파이프라인의 뒷 단계에서 앞 단계에서 사용된 값이 필요한 경우는 앞 단계의 연산을 역으로 수행하자
    - 예시: 메르센 소수 (2^p - 1 꼴의 숫자 중 p가 소수인 수)
    - 소수의 무한 스트림을 반환하는 `primes()` 메서드를 이용해 처음 20개의 메르센 소수를 구현하는 프로그램을 다음과 같이 구현할 수 있다.
    ```java
    public class MersennePrimes {
        static Stream<BigInteger> primes() {
            return Stream.iterate(TWO, BigInteger::nextProbablePrime);
        }

        public static void main(String[] args) {
            primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                    .filter(mersenne -> mersenne.isProbablePrime(50))
                    .limit(20)
                    .forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
        }
    }
    ```
    - 마지막 줄에서 p와 메르센 소수를 함께 구현하는데, 메르센 소수로부터 2^p-1을 역연산(비트 길이)하여 p를 구하였다.
- 애매한 경우
    - 예시: 카드 덱 초기화
    - 숫자와 무늬의 모든 가능한 조합을 만드는 문제 (데카르트 곱)
    ```java
    public class Card {
        public enum Suit { SPADE, HEART, DIAMOND, CLUB }
        public enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN,
                        EIGHT, NINE, TEN, JACK, QUEEN, KING }

        private final Suit suit;
        private final Rank rank;

        @Override public String toString() {
            return rank + " of " + suit + "S";
        }

        public Card(Suit suit, Rank rank) {
            this.suit = suit;
            this.rank = rank;

        }
        private static final List<Card> NEW_DECK = newDeck();

        // Iterative Cartesian product computation
        private static List<Card> newDeck() {
            List<Card> result = new ArrayList<>();
            for (Suit suit : Suit.values())
                for (Rank rank : Rank.values())
                    result.add(new Card(suit, rank));
            return result;
        }

    //    // Stream-based Cartesian product computation
    //    private static List<Card> newDeck() {
    //        return Stream.of(Suit.values())
    //                .flatMap(suit ->
    //                        Stream.of(Rank.values())
    //                                .map(rank -> new Card(suit, rank)))
    //                .collect(toList());
    //    }

        public static void main(String[] args) {
            System.out.println(NEW_DECK);
        }
    }
    ```
    - flatMap: 원소를 스트림으로 만든 다음 각 스트림들을 하나의 스트림으로 합치는 연산
    - 취향에 따라 선택하면 된다.