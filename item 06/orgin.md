# 불필요한 객체 생성을 피하라
```java
// Wrong
String s = new String("bikini");
// Correct
String s = "bikini";
````

- 생성자 대신 정적 팩터리 매서드([아이템 1](https://github.com/coukieStudy/Effective-Java/blob/master/Item%201/origin.md))를 제공하는 불변 클래스에서는 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있다.
```java
// Before
Boolean(String)
// static factory method
Boolean.valueOf(String)
````

- 생성 비용이 아주 비싼 객체도 더러 있다. 이런 ‘비싼 객체’가 반복해서 필요하다면 캐싱하여 재사용하길 권한다.
  - 당연하지만, 안 쓰면 캐시가 비효율적이 된다.
```java
public class RomanNumerals {
  private static final Pattern ROMAN = Pattern.compile(
    "^(?=.)M*(C[MD]|D?C{0,3})"
    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    
  static boolean isRomanNumeral(String s) {
      return ROMAN.matcher(s).matches(); 
    }
}
```
- keySet 예제?

- 오토박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아니다.
```java
private static long sum() {
  Long sum = 0L;
  for (long i = 0; i <= Integer.MAX_VALUE; i++) // How about "Long i = Long.valueOf(0);" ?
    sum += i;
  return sum;
}
```
