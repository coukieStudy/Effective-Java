# 최적화는 신중히 하라

#### 빠른 프로그램보다는 좋은 프로그램이 좋다.

좋은 프로그램인데 성능이 나오지 않을 경우, 아키텍쳐 자체가 최적화할 수 있는 길을 안내할 것이다. 정보 은닉 원칙을 따르면 개별 구성요소의 내부를 독립적으로 설계할 수 있다. 그렇지만 성능을 무시하라는 뜻은 아니다. 

#### 성능을 제한하는 설계를 피하라.

완성 후 변경하기 어려운 설계 요소는 컴포넌트끼리 또는 외부 시스템과의 소통방식이다. (API, 네트워크 프로토코르 영구 저장용 데이터 포맷 등) 이런 것들은 변경이 어렵고 성능을 제한할 수 있다.

#### API를 설계할 때 성능에 주는 영향을 고려하자.

- public 메소드에서 내부 데이터를 변경할 수 있게 만들면 불필요한 방어적 복사를 유발한다. (item 50)

- 또한 컴포지션으로 해결할 수 있는 public클래스를 상속으로 처리한다면 영원히 상위 클래스에 종속되고 성능까지 물려받는다. (item 18)

- 구현타입보다 인터페이스를 사용하라. (item  64)

#### 성능을 위해 API를 왜곡하는 건 매우 안 좋은 생각이다.

왜곡된 API를 지원하는 데 따르는 고통은 영원하다.

####  최적화 시도 전후로 성능을 테스트하라.

- 프로파일링 도구를 사용하자. 최적화 노력을 어디에 집중해야 할지 찾을 수 있다.
- 자바는 수행과 명령 사이에 추상화 격차가 커서 최적화로 인한 성능 변화를 일정하게 예측하기 어렵다.
- 사용하는 하드웨어 플랫폼마다 최적화 효과를 측정해야 한다.