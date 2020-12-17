# Item 47. 반환 타입으로는 스트림보다 컬렉션이 낫다



Stream은  Iterable을 확장하지 않으므로 반복을 지원하지 않는다. 따라서 for-each로 스트림을 반복할 수 없다.

Stream에 꼭 반복을 사용하고자하면 중개어댑터를 사용하는 방법이 있다.

```java
public static <E> Iterable<E> iterableOf(Stream<E> stream){
    return stream::iterator;
}

public static void main(String[] args) {
    for(ProcessHandle p : iterableOf(ProcessHandle.allProcesses())){
        //프로세스를 처리한다. 
    }
}
```

Iterable을 Stream으로 바꿔주는 중개어댑터는 아래와 같다.

```java
public static <E> Stream<E> streamOf(Iterable<E> iterable){
    return StreamSupport.stream(iterable.spliterator(), false);
}
```



Collection 인터페이스는 Iterable의 하위타입이고 stream 메서드도 제공하니 반복과 스트림을 동시에 지원한다. 따라서 **원소 시퀀스를 반환할 때 반환타입은 Collection이나 그 하위 타입을 사용하는게 일반적으로 최선이다.**

ex) Arrays => Arrays.asList, Stream.of 사용 가능

**하지만 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안된다!** 반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면 **전용 컬렉션**을 구현하자!

```java
public class PowerSet {
    public static final <E> Collection<Set<E>> of(Set<E> s) {
       List<E> src = new ArrayList<>(s);
       if(src.size() > 30) {
           throw new IllegalArgumentException("집합에 원소가 너무 많습니다(최대 30개).: " + s);
       }

       return new AbstractList<Set<E>>() {
           @Override
           public int size() {
               return 1 << src.size();
           }

           @Override
           public boolean contains(Object o) {
               return o instanceof Set && src.containsAll((Set) o);
           }

           @Override
           public Set<E> get(int index) {
               Set<E> result = new HashSet<>();
               for (int i = 0; index != 0; i++, index >>=1) {
                   if((index & 1) == 1) {
                       result.add(src.get(i));
                   }
               }
               return result;
           }
       };
    }
}
```



