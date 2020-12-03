## Item 36 비트 필드 대신 EnumSet을 사용하라

옛날에는 아래와 같은 코드를 썼었나보다..?

```java
public class Text {
  public static final int STYLE_BOLD = 1 << 0; // 1
  public static final int STYLE_ITALIC = 1 << 1; // 2
  public static final int STYLE_UNDERLINE = 1 << 2; // 4
  public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8
  
  // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
	public void applyStyles(int styles) { ... } 
}

text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

하지만 비트 필드는 정수 열거 상수의 단점을 그대로 지니며, 추가로 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해 석하기가 훨씬 어렵다. 비트 필드 하나에 녹아 있는 모든 원소를 순회하기도 까다롭다. 마지막으로, 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입(보통은 int나 long)을 선택해야 한다.



### java.util 패키지의 EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다

코드 36-2 EnumSet - 비트 필드를 대체하는 현대적 기법

```java
public class Text {
	public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
	
  // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
	public void applyStyles(Set<Style> styles) { ... } 
}

text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```


