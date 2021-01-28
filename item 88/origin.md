# readObject 메서드는 방어적으로 작성하라.

Period 객체의 물리적 표현과 논리적 표현이 부합하므로 기본 직렬화 형태를 사용해도 나쁘지 않다. 

```java
// 방어적 복사를 사용하는 불변 클래스
public final class Period {
    private final Date start;
    private final Date end;
    
    /**
     * @param start 시작 시각
     * @param end 종료 시각; 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발행한다.
     */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if(this.start.compareTo(this.end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
        }
    }
    
    public Date start() { return new Date(start.getTime()); }
    public Date end() { return new Date(end.getTime()); }
    public String toString() { return start + "-" + end; }
}
```

하지만 단지 `implements Serializable`을 사용하는 것은 불변식을 더이상 보장하지 못하게 한다. <br>



**문제 1)**

readObject 메서드를 작성할 때는 언제나 public 생성자를 작성하는 자세로 임해야 한다. (인수의 유효성 검사, 매개변수 방어적 복사 필요)

```java
public class BogusPeriod {
    private static final byte[] serializedForm = { // 가짜 바이트 스트림
        (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06....
    }
    
    public static void main(String[] args) {
        Period p = (Period) deserialize(serializedForm);
        System.out.println(p);
    }
    
    static Object deserialize(byte[] sf) {
        try {
            return new ObjectInputStream(new ByteArrayInputStream(sf)).readObject();
        } catch(IOException | ClassNotFoundException e) {
            throw new IllegalArgumentException(e);
        }
    }
}
```

**문제 2)** 

정상 Period 인스턴스에서 시작된 바이트 스트림 끝에  private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어낼 수 있다.<br>

**해결방법**<br>

Period의 readObject 메서드가 defaultReadObject를 호출한 다음 역직렬화된 객체가 유효한지 검사해야 한다.<br>

방어적 복사를 유효성 검사보다 앞서 수행하고 clone 메서드는 사용하지 않는다. 방어적 복사를 할 때는 final 한정자는 제거해야 한다. 아쉽지만 공격 위험 노출보다 낫다.

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    
    // 가변 요소들을 방어적으로 복사한다.
    start = new Date(start.getTime());
    end = new Date(end.getTime());
    
    // 불변식을 만족하는지 검사한다.
    if(start.compareTo(end) > 0) {
       throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```



### 기본 readObject 메서드를 써도 좋은 경우

- transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은가?
  - 아니오 ->커스텀 readObject 메서드를 만들어 유효성 검사와 방어적 복사를 수행
  - 예 -> 기본 readObject 메서드 사용



주의: final이 아닌 직렬화 가능 클래스라면 readObject 메서드도 재정의 가능 메서드를 직접적으로든 간접적으로든 호출해서는 안된다.
