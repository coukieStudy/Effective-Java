# 상속보다는 컴포지션을 사용하라


상속은 캡슐화를 깨트린다 - 상위 클래스의 구현에 따라 하위 클래스가 영향을 받을 수 있다.
예시) 
```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    // The number of attempted element insertions
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedHashSet<Integer> s = new InstrumentedHashSet<>();
        s.addAll(List.of(1, 2, 3));
        System.out.println(s.getAddCount());
    }
}
```
위 코드에서 s.getAddCount()의 결과는 3이 아니라 6이다. 그 이유는 HashSet.addAll 내부에서 add를 호출하는데, 이 때 InstrumentedHashSet.add가 호출되면서 원소의 개수만큼 addCount++이 호출되기 때문이다.  

Q. addAll() 자체를 재정의 하지 않으면 되지 않는가?  
A. HashSet의 addAll이 add를 이용했다는 가정 하에만 성립되는 해법이다. 상위 클래스의 코드의 구현에 의존하는 해법

Q. addAll() 을 다른 방식으로 재정의할 수 있지 않는가? 예를 들어 parameter의 원소의 수 만큼 Instrumented.add를 호출하는 방식이라든지...  
A. 상위 클래스의 동작을 다시 구현하는 방식은 비효율적이며 오류가 날 수도 있다. 뿐만 아니라 하위클래스에서는 접근할 수 없는 상위클래스의 private 필드를 써야한다면 구현이 불가능하다.

Q. 위의 상황들은 메서드 재정의가 원인이라 할 수 있다. 따라서 메서드 재정의를 하지 않고 아예 다른 메서드를 추가하면 안되나?  
A. 추가한 메서드랑 이름이 똑같은 메서드가 상위클래스에 추가되면 Compile Error 혹은 재정의에서 발생한 문제를 피할 수 없다. 뿐만 아니라 상위클래스에서 추가한 메서드의 규약을 지키지 못하게 된다.

뿐만 아니라 상위클래스에 새로운 메서드가 추가되면, 이를 고려하지 않은 하위클래스들에서 오류가 발생할 수 있다. 하위클래스들에서 특정 조건을 만족하는 Collection을 구현하고 있었는데, 이 컬렉션에 원소를 추가하는 함수가 새로 추가되면 하위클래스의 규약이 깨질 수 있다.

### 컴포지션 방식
***
컴포지션 방식을 쓰면 위에서 발생한 많은 문제를 해결할 수 있다.
1. 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 한다 -> Composition
2. 새 클래스의 인스턴스 메서드들은 (private 필드로 참조하는) 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다 -> Forwarding

```java
// 다른 메서드들 생략 
class InstrumentedHashSet<E> {
	private HashSet<E> s;
	private int addCount = 0;
  
	public InstrumentedHashSet(HashSet<E> hashSet) {
		this.s = hashSet;
	}

	public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return s.addAll(c);
	}

	public boolean add(E e) {
		addCount += 1;
		return s.add(e);
	}
}
```
* 책에 나온 코드 18-3 ForwardingSet을 사용하면 HashSet뿐 아니라 다른 Set에도 적용할 수 있다.

컴포지션 방식은 콜백 프레임워크와는 어울리지 않는다. 아래의 코드에서 Controller 를 통해 Model을 변경시키면, ModelChangesCounter에서는 이를 인지하지 못한다.
```java
// basic class which we will wrap
public class Model{ 
    Controller controller;

    Model(Controller controller){
        this.controller = controller; 
        controller.register(this); //Pass SELF reference
    }

    public void makeChange(){
        ... 
    }
} 

public class Controller{
    private final Model model;

    public void register(Model model){
        this.model = model;
    }

    // Here the wrapper just fails to count changes, 
    // because it does not know about the wrapped object 
    // references leaked
    public void doChanges(){
        model.makeChange(); 
    } 
}

// wrapper class
public class ModelChangesCounter{
    private final Model; 
    private int changesMade;

    ModelWrapper(Model model){
        this.model = model;
    }

    // The wrapper is intended to count changes, 
    // but those changes which are invoked from 
    // Controller are just skipped    
    public void makeChange(){
        model.makeChange(); 
        changesMade++;
    } 
}
```

상속은 is-a 관계가 명확히 성립할 때만 사용해야 한다. 스택은 백터가 아니기에 Stack은 Vector를 상속해서는 안됐고, 속성 목록도 해시테이블이 아니므로 Properties 가 HashTable을 상속하면 안됐다.  

is-a 관계가 성립하더라도, 상위 클래스에 결함이 있다면 상속을 사용했을 때 하위 클래스로 결함이 전파되므로 사용하면 안된다. 상위클래스가 확장을 고려해 설계된 클래스가 아닌 경우에도 상속은 문제가 될 수 있다.





