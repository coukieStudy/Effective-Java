# Chapter 17 - 변경 가능성을 최소화하라

## 불변 클래스

- 정의: 객체가 생성 후 파괴되는 순간까지 인스턴스의 내부 값을 수정할 수 없는 클래스
- 예시: String, Boxing Class(Integer, Double), BigInteger, BigDecimal

## 불변 클래스를 만들기 위한 5가지 규칙

1. 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
    - 예를 들면, setter
2. 클래스를 확장할 수 없도록 한다.
    - 하위 클래스에서 객체의 상태를 변하게 만드는 사태를 막아준다.
3. 모든 필드를 final로 선언한다.
4. 모든 필드를 private으로 선언한다.
5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
    - 가변 객체를 참조하는 필드가 있다면, 그 객체의 참조를 얻을 수 없도록 해야 한다.
    - 생성자, 접근자, readObject 메서드(Item 88) 모두에서 방어적 복사를 수행해라.

## 불변의 예제

- Item 11의 PhoneNumber

    ```java

    public final class PhoneNumber {
        private final short areaCode, prefix, lineNum;

        public PhoneNumber(int areaCode, int prefix, int lineNum) {
            this.areaCode = rangeCheck(areaCode, 999, "area code");
            this.prefix   = rangeCheck(prefix,   999, "prefix");
            this.lineNum  = rangeCheck(lineNum, 9999, "line num");
        }

        private static short rangeCheck(int val, int max, String arg) {
            if (val < 0 || val > max)
                throw new IllegalArgumentException(arg + ": " + val);
            return (short) val;
        }

        @Override public boolean equals(Object o) {
            if (o == this)
                return true;
            if (!(o instanceof PhoneNumber))
                return false;
            PhoneNumber pn = (PhoneNumber)o;
            return pn.lineNum == lineNum && pn.prefix == prefix
                    && pn.areaCode == areaCode;
        }

    		//Need Hashcode

        public static void main(String[] args) {
            Map<PhoneNumber, String> m = new HashMap<>();
            m.put(new PhoneNumber(707, 867, 5309), "Jenny");
            System.out.println(m.get(new PhoneNumber(707, 867, 5309)));
        }
    }
    ```

- Item 17의 Complex: 불변 복소수(실수부 + 허수부) 클래스
    - 함수형 프로그래밍: 아래 코드처럼 피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴.
        - 또한, add 같은 동사 대신에 plus 같은 전치사를 사용한 점도 주목. 이는 해당 메서드가 객체의 값을 변경하지 않는다는 사실을 강조하려는 의도.
        - 이러한 명명 규칙을 따르지 않은 BigInteger, BigDecimal 클래스를 사람들이 잘못 사용해 오류가 발생하는 일이 자주 있다. (add...)
    - 절차적, 명령형 프로그래밍: 메서드에서 피연산자인 자신을 수정해 자신의 상태가 변하게 된다.

    ```java
    package effectivejava.chapter4.item17;

    // Immutable complex number class (Pages 81-82)
    public final class Complex {
        private final double re;
        private final double im;

        public static final Complex ZERO = new Complex(0, 0);
        public static final Complex ONE  = new Complex(1, 0);
        public static final Complex I    = new Complex(0, 1);

        public Complex(double re, double im) {
            this.re = re;
            this.im = im;
        }

        public double realPart()      { return re; }
        public double imaginaryPart() { return im; }

        public Complex plus(Complex c) {
            return new Complex(re + c.re, im + c.im);
        }

        // Static factory, used in conjunction with private constructor (Page 85)
        public static Complex valueOf(double re, double im) {
            return new Complex(re, im);
        }

        public Complex minus(Complex c) {
            return new Complex(re - c.re, im - c.im);
        }

        public Complex times(Complex c) {
            return new Complex(re * c.re - im * c.im,
                    re * c.im + im * c.re);
        }

        public Complex dividedBy(Complex c) {
            double tmp = c.re * c.re + c.im * c.im;
            return new Complex((re * c.re + im * c.im) / tmp,
                    (im * c.re - re * c.im) / tmp);
        }

        @Override public boolean equals(Object o) {
            if (o == this)
                return true;
            if (!(o instanceof Complex))
                return false;
            Complex c = (Complex) o;

            // See page 47 to find out why we use compare instead of ==
            return Double.compare(c.re, re) == 0
                    && Double.compare(c.im, im) == 0;
        }
        @Override public int hashCode() {
            return 31 * Double.hashCode(re) + Double.hashCode(im);
        }

        @Override public String toString() {
            return "(" + re + " + " + im + "i)";
        }
    }
    ```

## Thread Safe

- 불변 객체는 근본적으로 Thread-Safe하여 따로 동기화할 필요 없다. Read-Only이기 때문에
- 따라서, 불변 클래스라면 한번 만든 인스턴스를 최대한 재활용하기를 권한다.
    - 가장 쉬운 재활용 방법은 아래처럼 public static final로 제공하는 것

        ```java
        public static final String title = "1111"
        ```

    - 정적 팩터리 제공(Item 01 참고): 메모리 사용량과 가비지 컬렉션 비용이 줄어든다.

        ```java
        public static Boolean valueOf(boolean b){
          return b ? Boolean.TRUE: Boolean.FALSE;
        }
        ```

## 불변 객체의 특징

- 불변 객체를 자유롭게 공유할 수 있다는 것은 방어적 복사(Item 50)도 필요 없기에... 불변 객체는 clone 메서드나 복사 생성자를 제공하지 않는 것이 좋다.
- 불변 객체끼리 내부 데이터를 공유할 수 있다.(사실 똑같은 말임..)
    - 예시: BigInteger 클래스는 값의 부호(sign)와 크기(magnitude)를 따로 표현한다. 이를 위해, 부호에 int 변수, 크기에 int 배열을 사용한다.

        ```java
        final int signum; //부호
        final int[] mag;  //크기

        /* 
         * negate 함수: 크기가 같고 부호만 반대인 새로운 Integer 생성 
         * 내부 배열(this.mag)을 그대로 사용
         */
        public BigInteger negate() {
            return new BigInteger(this.mag, -this.signum);
        }
        ```

- 불변 객체는 Map의 키나 Set의 원소로 쓰기에 안성맞춤이다. Map이나 Set은 안에 담긴 값이 바뀌면 불변식이 허물어지는데, 불변 객체를 사용하면 그런 걱정은 하지 않아도 된다.

## 불변 객체의 단점

- 값이 다르면 반드시 독립된 객체로 만들어야 한다는 것.
- 예를 들어, 백만 비트짜리 BigInteger에서 비트 하나를 바꿔야 한다고 할 때 flipBit 메서드는 새로운 BigInteger 인스턴스를 생성한다.

```java
/* 
 * flipBit는 N + 1번째 bit를 전환하는 함수 
 * paramter N이 0일 경우, 1111 -> 1110
 */
BigInteger moby = ...;
moby = moby.flipBit(0);
```

- 이런 경우 가변인 bitSet을 사용하여 성능을 끌어올릴 수 있다.

## Companion Class - 가변 동반 클래스

- 위에서 더 나아가서 원하는 객체를 완성하기까지 많은 단계가 존재하고, 해당 단계마다 만들어진 객체들이 모두 버려진다면 성능 문제가 더욱 불거진다.
- 해결책
    - multistep operation(다단계 연산)들을 예측하여 기본 기능으로 제공
        - 이렇게 다단계 연산을 기본으로 제공한다면 단계마다 객체 생성을 하지 않아도 되어 성능 문제가 없어진다. 아래처럼 Companion Class를 package-private으로 두고 내부에서 구현하게 되면, 사용하는 입장에서는 매우 편리하다.
        - BigInteger의 모듈러 지수을 Companion Class로서 package-private으로 두고 있다.
        - String은 StringBuilder와 StringBuffer를 Companion Class로서 package-private으로 두고 있다.

        [String, StringBuffer, StringBuilder의 차이점과 장단점은 뭔가요?](https://www.slipp.net/questions/271)

        - 위 글을 따라서 읽어보면, String에서 StringBuilder를 자동으로 지원해준다고 해서  +를 남발하지 말고.. 아래의 조건에 따라 사용하면 좋을 듯

        ```java
        //정적으로 유지하는 경우 사용이 용이
        //디컴파일해보면, +부분이 모두 StringBuilder로 append한다.
        final String aaa = "a" + "b" + "c"; 

        //런타임 시에 바뀌는 경우 StringBuilder.
        StringBuilder cccc...
        ```

## 상속하지 못하도록 하는 방법

1. 가장 쉬운 법은 final 클래스로 선언
2. 더 유연한 방법은 모든 생성자를 private 혹은 package-private으로 만들고 public 정적 팩터리를 제공하는 방법
    - package-private으로 지정한다면, 원하는 만큼 패키지 내부에서 만들어 사용 가능하기에 더욱 유연
- 하지만 BigInteger나 BigDecimal는 상속하지 못하도록 못했기 때문에, 신뢰할 수 없는 하위 클래스의 인스턴스라고 확인되면, 이 인수들은 가변이라 가정하고 방어적으로 복사해야 한다. (외부 라이브러리...등등)

## 너무 심한 조건 모두 final!

- 모두 final은 너무 심한 조건이었기 때문에.. **외부에 나타나는 값을 변경할 수 없다** 정도로 바꿀 수 있겠다.
    - 이게 무슨 말이냐면, 내부에서 계산 비용이 큰 값들은 final이 아닌 곳에 캐싱해서 사용할 수 있다는 이야기

    ```java
    /* Item 11의 PhoneNumber의 hashcode */
    private int hashCode; // 0으로 자동으로 초기화
    @Override
    public int hashCode() {
    	int result = hashCode;
    	if (result == 0) { 
    		/* update result using new hashCode logic */ 
    	}
    	return result;
    }
    ```

### 무작정 불변 지양

- 이후에 변경 가능성 등등 고려

### 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자
