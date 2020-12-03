## Item 37 ordinal 인덱싱 대신 EnumMap을 사용하라

```java
class Plant {
	enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

  final String name;
	final LifeCycle lifeCycle;

  Plant(String name, LifeCycle lifeCycle) {
    this.name = name;
		this.lifeCycle = lifeCycle;
  }
	
  @Override
  public String toString() {
		return name; 
  }
}
```



###### 코드 37-1 **ordinal()**을 배열 인덱스로 사용 - 따라 하지 말 것!

```java
Set<Plant>[] plantsByLifeCycle =
  (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for (int i = 0; i < plantsByLifeCycle.length; i++)
  plantsByLifeCycle[i] = new HashSet<>();

for (Plant p : garden)
  plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);

// 결과 출력
for (int i = 0; i < plantsByLifeCycle.length; i++) {
	System.out.printf("%s: %s%n", 
                    Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

배열은 제네릭과 호환되지 않으니(아이템 28) 비검사 형변환을 수행해야 하고 깔끔히 컴파일되지 않을 것

배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야함

**정확한 정숫값을 사용한다는 것을 개발자가 직접 보증해야 함**



열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현체가 존재하는 데, 바로 EnumMap

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
for (Plant.LifeCycle lc : Plant.LifeCycle.values())
  plantsByLifeCycle.put(lc, new HashSet<>());
for (Plant p : garden) 
  plantsByLifeCycle.get(p.lifeCycle).add(p);
System.out.println(plantsByLifeCycle);
//ANNUAL=[e, a], PERENNIAL=[b, c], BIENNIAL=[d]
```

스트림을 쓰면 아래와 같이 좀 더 단순하게 구현할 수 있는데

```java
System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle)));
```

이 코드는 EnumMap이 아닌 고유한 맵 구현체를 사용했기 때문에 EnumMap을 써 서 얻은 공간과 성능 이점이 사라진다는 문제가 있다.

아래와 같이 최적화가 가능하다.

```java
System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle, () -> new EnumMap<>(LifeCycle.class), toSet())));
```

매개변수 3개짜리 Collectors.groupingBy 메서드는 mapFactory 매개변수에 원하는 맵 구현체를 명시해 호출할 수 있다.

**배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니, 대신 Enum Map을사용하라.**
