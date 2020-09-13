# Item05. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라.

> ### 클래스가 하나 이상의 자원에 의존하고 사용하는 자원에 따라 동작이 달라지는 경우, <br> 정적 유틸리티 클래스나 싱글턴 방식은 적합하지 않다. <br> 대신 ✨의존 객체 주입 기법✨을 사용하자!
<br>

## 자원을 직접 명시하는 경우 🤔

### 정적 유틸리티 클래스 사용 ([Item04](https://github.com/coukieStudy/Effective-Java/blob/master/item%204/origin.md))

```java
public class SpellChecker {
	private static final Lexicon dictionary = ...;
	private SpellChecker() {} // 객체 생성 방지
	public static boolean isValid(String word) { ... }
	public static List<String> suggestions(String typo) { ... }
}
```

### 싱글턴 사용 ([Item03](https://github.com/coukieStudy/Effective-Java/blob/master/item%203/origin.md))

```java
public class SpellChecker {
	private final Lexicon dictionary = ...;
	private SpellChecker(...) {}
	public static SpellChecker INSTANCE = new SpellChecker(...);
	public boolean isValid(String word) { ... }
	public List<String> suggestions(String typo) { ... }
}
```

### 문제점

- 위의 경우, 여러 사전을 사용할 수 없다. → 유연성 X
- *final* 을 제거하고 사전을 교체하는 메소드 추가시, 여러 사전을 사용할 수는 있지만 오류 내기 쉽고 멀티스레드 환경에서 사용할 수 없다.

>
```java
public class SpellChecker {
	private Lexicon dictionary;
	public void setDictionary(Lexicon dictionary) { this.dictionary = dictionary; }
	public boolean isValid(String word) { ... }
	public List<String> suggestions(String typo) { ... }
}
```
- 테스트하기 어렵다.

<br>

## 의존 객체 주입 기법 🤗

### 인스턴스를 생성할 때 생성자에 자원을 넘겨주는 방식

```java
public class SpellChecker {
	private final Lexicon dictionary;
	public SpellChecker(Lexicon dictionary) {
			this.dictionary = Objects.requireNonNull(dictionary);
	}
	public boolean isValid(String word) { ... }
	public List<String> suggestions(String typo) { ... }
}
```

### 생성자에 자원 팩터리를 넘겨주는 방식

```java
public SpellChecker(Supplier<? extends Lexicon> dicFactory) {
	this.dictionary = dicFactory.get(); 
}
```

위와 같이 Supplier를 사용해서 명시한 타입의 하위 타입을 생성할 수 있는 팩터리를 넘길 수 있다.

### 특징

- 유연성과 테스트 용이성 개선
- 의존성이 많아지고 프로젝트가 커지면 코드가 복잡할 수 있다. → 의존 객체 주입 프레임워크 사용해서 해결.(Dagger, Guice, Spring)
