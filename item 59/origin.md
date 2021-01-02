# 라이브러리를 익히고 사용하라

## 예시: Random
- 주어진 값의 범위 내에서 무작위 정수 하나를 생성하는 메서드를 만든다고 해보자. 대부분 다음과 같이 만들 것이다.
```java
static Random rnd = new Random();

static int random(int n) {
    return Math.abs(rnd.nextInt()) % n;
}
```
- 이 메서드는 멀쩡해 보이지만, 실제로는 세 가지 문제를 가지고 있다.
    - n이 크지 않은 2의 제곱수라면 얼마 지나지 않아 같은 수열이 반복된다.
    ```java
    protected int next(int bits) {
        long oldseed, nextseed;
        AtomicLong seed = this.seed;
        do {
            oldseed = seed.get();
            nextseed = (oldseed * multiplier + addend) & mask; // nextseed 값은 mask에 의존적이다.
        } while (!seed.compareAndSet(oldseed, nextseed));
        return (int)(nextseed >>> (48 - bits));
    }
    ```
    - n이 2의 제곱수가 아니라면 몇몇 숫자가 평균적으로 더 많이 반환된다. n이 클수록 더 크렇다.
        - int에는 범위가 있고, 범위의 꼬리 부분은 n으로 나눈 모든 경우를 커버하지 못할 수 있다.
    - 지정한 범위 바깥의 수가 종종 반환될 수 있다.
        - abs(n) = n ^ 0 + 1인데, 10000000 ^ 00000000 + 00000001 = 10000000
- 이미 만들어져 있는 `Random.nextInt(int)`를 사용하는 게 좋다. 전문가들과 사용자들에 의해 검증된 안전한 메서드이다.
- 자바 7부터는 `ThreadLocalRandom`을 사용하는 것이 좋다. `Random`보다 더 무작위적이고 더 빠르다.

## 표준 라이브러리의 이점
1. 전문가들과 다른 개발자들에 의해 검증된 것이다.
2. 내 프로젝트에 집중할 수 있다.
3. 가만히 있어도 성능이 개선된다.
4. 가만히 있어도 기능이 많아진다.
5. 다른 사람들이 읽기 편해진다.

- 이렇게 장점이 많은데 실제로 개발자들은 잘 안 쓴다. 왜? 몰라서.
- 자바는 메이저 릴리즈마다 좋은 기능들이 라이브러리에 추가된다.
    - https://www.oracle.com/java/technologies/javase/15-relnote-issues.html
    - 예시: 특정 URL의 내용을 가져오는 애플리케이션
    ```java
    public static void main(String[] args) throws IOException {
        try (InputStream in = new URL(args[0]).openStream()) {
            in.transferTo(System.out);
        }
    }
    ```
    - `transferTo`가 없었다면?: https://stackoverflow.com/questions/43157/easy-way-to-write-contents-of-a-java-inputstream-to-an-outputstream
- 라이브러리가 엄청 많긴 하지만, 최소한 다음 라이브러리들은 공부해두자.
    - `java.lang`
    - `java.util` (특히 collection, stream, concurrent)
    - `java.io`
- 자바 표준 라이브러리 > 서드파티 라이브러리 > 직접 구현