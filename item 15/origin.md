# 15. 클래스와 멤버의 접근 권한을 최소화하라.


**정보 은닉이란? 내부 데이터와 내부 구현 정보를 외부 컴포넌트로부터 숨기는 것**

<br>

## 정보 은닉의 장점

**컴포넌트의 독립성을 높여 개별적 개발, 테스트, 최적화, 적용, 분석, 수정 가능**

- 병렬 개발 가능 -> 개발 속도 ⬆
- 다른 컴포넌트로 교체 부담 ⬇
- 해당 컴포넌트의 최적화에 집중 가능
- 독자적 동작이 가능 -> 재사용성 ⬆, 개별 컴포넌트 동작 검증 쉬움


<br>

## 접근 제한자의 활용

**올바르게 동작하는 한 가장 낮은 접근 수준을 부여해야 한다.**

- public을 사용하면 공개 API가 되고, 하위 호환을 위해 영원히 관리해줘야 한다.
-  따라서 패키지 외부에서 쓸 이유가 없다면 public보다 package-private로 선언하자. 
- 한 클래스에서만 사용하는 package-private 톱레벨 클래스나 인터페이스는 이를 사용하는 클래스 안에 private static으로 중첩시켜보자. (Item 24 참고)
- private와 package-private 멤버는  Serializable로 구현한 클래스에서는 의도치 않게 공개 API가 될 수 있다. (Item 86, 87 참고)
- public 클래스의 protected 멤버는 공개 API이다. 따라서 protected 멤버의 수는 적을수록 좋다.
- 상위 클래스의 메서드를 재정의할 때, 그 접근 수준을 상위 클래스보다 좁게 설정할 수 없다는 제약이 있다. (ex) 인터페이스 메서드 구현
-  테스트 코드만을 위해 공개 API로 만들지 말자.
- public 클래스의 인스턴스 필드는 되도록 public이 아니여야 한다.

	a. 그 필드의 불변을 보장 불가 <br>
	b. thread safe하지 않다. <br>
	c. 내부구현을 바꾸는 것에 문제 발생.<br>
	
- public static final 필드는 공개는 가능하다. 단 이 필드는 반드시 기본 타입 값이나 불변 객체를 참조해야 한다.

	a. public static final 배열 필드를 두지 말자. 배열 내의 변경이 가능하다. <br>	b. a의 해결책  1 : 불변리스트 사용<br>
	
	```java
	private static final Thing[] PRIVATE_VALUES = {...};
	public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
	```
	
	c. a의 해결책  2 : 방어적 복사<br>
	
	```java
	private static final Thing[] PRIVATE_VALUES = {...};  
	public static final Thing[] values() {  
		return PRIVATE_VALUES.clone();	
	}
	```

<br>

## Java 9 모듈 시스템

**모듈? 패키지들의 묶음<br>**

**모듈과 접근 제한**

- 모듈은 자기에게 속한 패키지 중 공개(export)할 것들을 따로(module-info.java) 선언한다.
- protected, public이라도 해당 패키지를 공개하지 않았다면 모듈 외부에서 접근 불가능.
- 하지만 모듈의 jar 파일을 자신의 모듈 경로가 아닌 애플리케이션의 classpath에 두면 그 모듈 안에 있는 패키지는 모듈이 없는 것처럼 행동한다.

**자바9 모듈 관련 참고 자료**

- [Java Jigsaw?- 자바9의 핵심] (https://greatkim91.tistory.com/197)
- [Project Jigsaw: Module System Quick-Start Guide] (https://openjdk.java.net/projects/jigsaw/quick-start)


