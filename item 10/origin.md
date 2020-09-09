# equals는 일반 규약을 지켜 재정의하라

#### 열거한 상황 중 하나에 해당한다면 재정의하지 않는게 최선이다.

- 각 인스턴스가 본질적으로 고유하다.

- 인스턴스의 ‘논리적 동치성(logical equality)’을 검사할 일이 없다.

- 상위 클래스에서 재정의한 **equals**가 하위 클래스에도 딱 들어맞는다.

- 클래스가 private이거나 package-private이고 **equals** 메서드를 호출할 일이 없다.

  ```java
  @Override public boolean equals(Object o) {
    throw new AssertionError(); // 호출 금지!
  }
  ```



#### equals 메서드를 재정의할 때는 반드시 일반 규약을 따라야 한다

- 반사성(reflexivity): x.equals.(x) == true {x != null}

- 대칭성(symmetry): x.equals(y) == y.eqauls(x) {x != null && y != null}

- 추이성(transitivity): if (x.equals(y) && y.equals(x)) x.equals(z) == true {x != null && y != null && z != null}

- 일관성(consistency): x.equals(y) == x.equals(y) {x != null && y != null}

- **null**-아님: x.equals(null) == false {x != null}

  

__수학에 익숙하지 않다면 이 내용이 조금 어려울 수 있다.__

- [동치류(equivalence class)](http://www.ktword.co.kr/abbr_view.php?m_temp1=4958)



__**equals** 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다.__

__구체 클래스를 확장해 새로운 값을 추가하면서 **equals** 규약을 만족시킬 방법은 존재하지 않는다.__

 - "상속 대신 컴포지션을 사용하"면 우회할 수 있다. (item18)



AutoValue framework을 사용하거나 IDE를 사용해서 equals를 만드는 것이 좋다.



**결론 : equals를 재정의하지 말자. 꼭 필요한 경우가 아니라면**
