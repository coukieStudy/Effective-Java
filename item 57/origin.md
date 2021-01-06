# 지역번수의 범위를 최소화하라

지역변수의 범위를 줄이는 가장 강력한 원칙: 가장 처음 쓰일 때 선언한다. 사용하려는 변수를 미리 선언해두면 가독성이 떨어지며 초기값 설정에서 오류가 날 수도 있다.

모든 지역변수는 선언과 동시에 초기화해야 한다. 단 예외는 try-catch 문이다. 사용하려는 변술르 try 블록 바깥에서도 사용해야 한다면, 정확히 초기화하지 못하더라도 try블록 앞에서 선언해야 한다.

반복문에서는 반복변수의 범위가 반복문의 몸체, 그리고 for 키워드의 괄호 안으로 제한된다. 따라서 반복 변수의 값을 이후에도 계속 쓰지 않는다면, while 보다 안전하다. while 문은 이 점 때문에 반복 변수의 값에 주의해야 한다.
for / for-each 문은 이러한 오류를 컴파일 시점에 잡아줄 수 있다.

```java
Iterator<Element> i = c.iterator(); while (i.hasNext()) {
  doSomething(i.next());
}
...
Iterator<Element> i2 = c2.iterator();
while (i.hasNext()) { // 버그!
  doSomethingElse(i2.next());
}
```

메서드를 작게 유지하고 한 가지 기능에 집중할 수 있도록 설계하는 것이 지역 변수 범위를 최소화하는 방법이다. 한 메서드에서 여러 기능을 처리하면, 한 기능과 관련된 지역변수라도 다른 기능을 수행하는 코드에서 접근할 수 있는데, 메서드를 기능별로 분할하면 이런 문제를 해결할 수 있다.