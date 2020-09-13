# toString을 항상 재정의해라

- Object toString의 default 반환값인 '클래스_이름@16진수로_표시한_해시코드'를 반환하지 말고, 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환하도록 하자.
  * ```java getClass().getName() + '@' + Integer.toHexString(hashCode()) ```

### toString을 재정의하면 좋은 경우
- 디버깅하는 경우
  * 객체가 거대하거나 객체의 상태가 문자로 표현하기 어렵다면, 요약 정보를 담아야 한다.
```java
{Jenny=PhoneNumber@adbbd}
{Jenny=707-867-5309} //권장: 유용한 메세지를 전달해줄 수 있다.
```
- 문서화하는 경우: toString의 반환값을 문서화하려는데 사용하는 경우
  * 전화번호나 행렬 같은 값 클래스라면 문서화하기를 권한다.
  * 포맷을 명시한다면 입출력/CSV 파일등으로 사림이 읽을 수 있는 데이터 포맷으로 저장할 수 있다. 이 경우 명시한 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩터리나 생성자를 함꼐 제공하면 좋다.
```java
//TODO: BigInteger, BigDecimal


```
  * 다만, 포맷을 명시하면 평생 그 포맷에 얽메이게 되어 유연성이 떨어짐.

#### toString이 반환한 값이 포함된 정보를 얻어올 수 있는 API를 제공하자.
- getter와 같은 것을 제공해서 toString 안에 포함된 각각 정보를 쉽게 가져올 수 있도록 해야 한다는 뜻. 안 그러면 toString을 파싱해야함으로 성능이 떨어지고, 필요하지 않은 작업!

#### 정적 유틸리티 클래스나 열거 타입은 toString을 따로 재정의할 필요 없다.
- name method를 사용하면 해당 enum 값의 이름을 가져올 수 있다.
- 평소에는 name method나 enum의 다른 필드 getter를 통해서 문자를 사용하기에 필요없다고 생각할 수 있지만, 좀 더 원하는 형식이 있을 경우 toString을 사용할 수 있는 있을 것 같다.
- 또한 하위 클래스들이 공유해야 할 문자열 표현이 있는 추상 클래스(AbstractMap, AbstractSet)라면 toString을 재정의하는 것이 좋다.
![스크린샷 2020-09-13 오후 11 06 15](https://user-images.githubusercontent.com/26040955/93020006-bcc51480-f615-11ea-8dc7-97c11b43203d.png)

```java
//AbstractMap의 toString
public String toString() {
    Iterator<Entry<K,V>> i = entrySet().iterator();
    if (! i.hasNext())
        return "{}";

    StringBuilder sb = new StringBuilder();
    sb.append('{');
    for (;;) {
        Entry<K,V> e = i.next();
        K key = e.getKey();
        V value = e.getValue();
        sb.append(key   == this ? "(this Map)" : key);
        sb.append('=');
        sb.append(value == this ? "(this Map)" : value);
        if (! i.hasNext())
            return sb.append('}').toString();
        sb.append(',').append(' ');
    }
}
```

#### AutoValue는 자동으로 잘 생성해주는 프레임워크이지만, '의미'까지 파악해서 생성해주지는 못한다.

- AutoValue: https://www.baeldung.com/introduction-to-autovalue
  

