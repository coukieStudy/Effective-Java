# 11. equals를 재정의하려거든 hashCode도 재정의하라.

## 1. hashCode 규약 [👀](https://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#hashCode())

- equals 비교에 사용되는 정보가 변경되지 않으면, application이 실행되는 동안 그 객체의 hashCode method는 항상 같은 값을 반환해야 한다. (단, application을 다시 실행할 경우는 달라져도 상관없다.)
- equals(Object)가 두 객체를 같다고 판단했으면 두 객체의 hashCode()는 똑같은 값을 반환해야 한다.
- equals(Object)가 두 객체가 다르다고 판단했어도 hashCode()값은 다를 필요가 없다. 하지만 다른 값을 반환해야 hash table의 성능이 좋아진다.

<br>

### 논리적으로 같은 객체는 같은 hashCode를 반환해야 한다.

```java
/*
* PhoneNumber의 equals를 areaCode, prefix, lineNum 
* 세 개가 전부 같아야만 true를 반환하도록 재정의한 경우.
*/
public static void main(String []args){
    Map<PhoneNumber, String> m = new HashMap<>();
    m.put(new PhoneNumber(707, 867, 5309), "제니");
    

    System.out.println(
        new PhoneNumber(707, 867, 5309).equals(new PhoneNumber(707, 867, 5309))); // true
    
    System.out.println(m.get(new PhoneNumber(707, 867, 5309))); // null

    System.out.println(new PhoneNumber(707, 867, 5309) == new PhoneNumber(707, 867, 5309)); 
		// false - 객체의 주소값으로 판단(System.identityHashCode(PhoneNumber pn)
 }
```

위의 경우, PhoneNumber 클래스는 equals의 재정의에 의해 논리적으로는 동치이지만 hashCode()를 재정의하지 않아서 "[규약 2] equals(Object)가 두 객체를 같다고 판단했으면 두 객체의 hashCode()는 똑같은 값을 반환해야 한다."를 만족하지 못한다.

<br>

### 적법하지만 최악의 hashCode 구현

```java
@Override
public int hashCode() { return 92; }
```

모든 객체가 hash table의 같은 버킷 하나에 존재해 linked list처럼 동작한다

→  hash Table의 평균 수행 시간이 O(1)에서 O(n)으로 느려질 수 있다.

[규약3] equals(Object)가 두 객체가 다르다고 판단했어도 hashCode()값은 다를 필요가 없다. 하지만 다른 값을 반환해야 hash table의 성능이 좋아진다.

<br><br>

## 2. hashCode 작성 요령

1. int result = c 로 초기화
2-3 반복적으로 수행.
2. equals 비교에 관여하는 모든 핵심 필드에 대해 아래 작업을 수행한다.

    a. 기본 타입 필드 → Type.hashCode(f)를 수행한다. (Type은 해당 기본 타입의 박싱 클래스)

    b. 참조 타입 필드 &&  equals가 이 필드를 재귀적으로 호출 → hashCode도 재귀적으로 호출.

    (만약 너무 계산이 복잡해지면 필드의 표준형을 만들어 호출한다. 필드가 null이면 0을 사용한다.)

    c. 필드가 배열 → 핵심 원소 각각을 별도 필드처럼 다뤄서 핵심 원소의 hashCode를 계산하고 3을 수행.

    (핵심 원소가 없으면 0 추천. 모든 원소가 핵심 원소이면 Arrays.hashCode 사용.)

3. result = 31 * result + c;
4. return result;

**<예시>**

```java
@Override
public int hashCode() {
	int result = Short.hashCode(areaCode);
	result = 31 * result + Short.hashCode(prefix);
	result = 31 * result + Short.hashCode(lineNum);
	return result;
}
```
<br><br>

## 3. 참고

- Item10 AutoValue 프레임워크를 사용하면 equals와 hashCode를 자동으로 잘 만들어준다.
- 해시 충돌이 더 적은 방법을 꼭 써야한다면 구아바[Guava]의 com.google.common.hash.Hashing을 참고해라.
- Objects.hash() : 임의의 개수만큼 객체를 받아 hashCode를 계산한다. 속도는 더 느리다.
ex) return Objects.hash(areaCode, prefix, lineNum);
- 클래스가 불변이고 해시코드 계산 비용이 크다면 매번 새로 계산하기보다 캐싱을 고려하자.

    ```java
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

- 성능을 높이기 위해서 핵심필드를 생략해서는 안된다.
