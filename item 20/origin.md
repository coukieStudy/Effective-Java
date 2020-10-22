# Chapter 20 추상 클래스보다는 인터페이스를 우선하라

## 인터페이스 vs 추상 클래스

- 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다. 따라서 새로운 인터페이스를 중간에 끼워넣는 것이 가능하다.

```java
public class sampleClass implement Comparable, Iterable //추가하면 됨
```

- 하지만 기존 클래스 위에 새로운 추상 클래스를 끼워넣기는 어려운 게 일반적이다. 두 클래스가 같은 추상 클래스를 확장하길 원한다면, 계층 구조상 같은 조상이어야 한다. 하지만 그렇게 만드는 것이 쉬운 일도 아니고 혼란을 야기할 수도 있다.

## Mixin 정의

- 믹스인이란? 대상 타입의 주된 기능에 선택적 기능을 '혼합(mixin)'하는 것.
- 예를 들어, Comparable을 추가하면 자신을 구현한 클래스의 인스턴스끼리는 순서를 정할 수 있다고 선택적 기능을 제공하는 것.

## 인터페이스 - 계층 구조가 없는 것 가능

- 타입을 계층적으로 정의하면 수많은 개념을 구조적으로 잘 표현할 수 있지만, 현실에는 계층을 엄격히 구분하기 어려운 경우도 많다.
    - 가수와 작곡자를 동시에 하는 경우도 있으니, 둘 다 상속받아서 구현해도 되고, 아래처럼 새로운 interface를 만들어도 된다.

```java
//가수
public interface Singer {
	AudioClip sing(Song s); 
}

//작곡자
public interface Songwriter {
	Song compose(int chartPosition); 
}

//가수이자 작곡자
public interface SingerSongwriter implements Singer, Songwriter { 
	AudioClip strum();
	void actSensitive(); 
}
```

- 위를 클래스로 표현하기는 어렵지만, 속성이 n개일 경우 조합을 계층 구조로 정의할 때 2의 n승이 된다.

## 인터페이스 예시

1. 래퍼 클래스(Item 18)과 함께 사용하면 Interface는 기능을 향상시키는 안전하고 강력한 수단이 된다.
    - 타입을 추상 클래스로 정의해두면 그 타입에 기능을 추가하는 방법은 상속뿐이다. 상속해서 만든 클래스는 래퍼 클래스보다 활용도가 떨어지고 깨지기는 더 쉽다.
2. 인터페이스 메소드 중 구현 방법이 명백한 것이 있다면, default를 이용해 구현할 수 있다.
    - 디폴트 메서드를 제공할 때는 @impleSpec 자바독 태그를 붙여 문서화(item 19)
    - ex) 21 - 1 removeIf
    - 당연히 Object의 equals, hashcode 같은 걸 default로 구현하면 안된다.
3. 인스턴스 필드 X, public이 아닌 정적 멤버도 가질 수 없다.(private 정적 메소드는 예외)
    - [https://beomseok95.tistory.com/272](https://beomseok95.tistory.com/272)
4. 인터페이스와 추상 skeletal 클래스를 함께 제공하는 식
    - [템플릿 메서드 패턴](https://www.slipp.net/wiki/display/SLS/Template+Method)
        - 예시) AbstractMap과 Map <br>
         ![_2020-10-21__11 55 39](https://user-images.githubusercontent.com/26040955/96865008-4df49a00-14a4-11eb-95f1-f2f5106d766e.png)

        - 인터페이스로는 타입을 정의하고, Skeletal class로는 나머지 메서드까지 구현한다. 이렇게 해두면 단순히 골격 구현을 확장하는 것만으로 이 인터페이스를 구현하는데 필요한 일이 대부분 완료.

    ```java
    public abstract class AbstractMap<K,V> implements Map<K,V>

    public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable
    ```

    - AbstractMap이 Map을 implement한 것임에도 HashMap이 Map을 상속한 이유?
        - 가독성을 올리기 위해서!
        - [https://stackoverflow.com/questions/11175058/why-does-hashmap-implement-map-if-it-extends-abstractmap](https://stackoverflow.com/questions/11175058/why-does-hashmap-implement-map-if-it-extends-abstractmap)

## Simulated multiple inheritance

- 다중 상속의 많은 장점은 제공하는 동시에 단점은 피하게 한다.
![_2020-10-22__12 01 29](https://user-images.githubusercontent.com/26040955/96865100-6cf32c00-14a4-11eb-9ba5-afd16c4ad31d.png)


## 핵심 정리

- 인터페이스 유용하다. 잘 쓰자.
- Mixin 용도로 많이 쓰임.
- 템플릿 메소드 패턴에서는 추상클래스와 인터페이스를 같이 사용하는 경우도 있다. (AbstractMap과 Map)
    - 예시!! → 내가 했던 것
