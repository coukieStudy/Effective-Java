# 전통적인 for문 보다는 for-each 문을 사용하라

전통적인 for문
```java
//반복자로 컬렉션 순회
for (Iterator<Element> i = c.iterator() ; i.hasNext() ; ) {
  Element e = i.next();
  ... //
}

//배열 순회
for (int i = 0 ; i < a.length ; i++) {
  ... // do something with a[i]
}
```

* 반복자나 인덱스의 등장 횟수가 많아서 변수를 잘못 사용할 여지가 많아진다.
* 컬렉션인지, 배열인지에 따라서 코드 형태가 달라진다.

이러한 단점은 for-each 문을 사용하면 모두 해결된다.

for each문은 Iterable 인터페이스를 구현한 객체라면 순회가 가능하다. 전통적인 for문과 성능의 차이도 없다.

중첩된 for문의 사용에서 for-each문의 장점은 더욱 부각된다.


```java
public class Card {
    private final Suit suit;
    private final Rank rank;

    enum Suit { CLUB, DIAMOND, HEART, SPADE }
    enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
        NINE, TEN, JACK, QUEEN, KING }

    static Collection<Suit> suits = Arrays.asList(Suit.values());
    static Collection<Rank> ranks = Arrays.asList(Rank.values());

    public static void main(String[] args) {
        List<Card> deck = new ArrayList<>();
        
        // NoSuchElementException 발생
        for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
            for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
                deck.add(new Card(i.next(), j.next()));
        
        // 이렇게 사용하는 것이 좋다.
        for (Suit suit : suits)
            for (Rank rank : ranks)
                deck.add(new Card(suit, rank));
    }
}
```

단 for-each 문을 사용할 수 없는 경우가 존재한다
1. 파괴적인 필터링: 컬렉션을 순회하면서 선택된 원소를 제거해야 하는 경우에는 반복자의 remove 메서드를 호출해야 한다.
2. 변형: 리스트나 배열을 순회하면서 그 원소의 값 일부, 혹은 전체를 교체해야 한다면 반복자나 배열의 인덱스를 사용해야 한다.
3. 병렬 반복: 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자나 인덱스 변수를 엄격하고 명시적으로 제어해야 한다.

