# 프로그램의 동작을 스레드 스케줄러에 기대지 말라

- 스레드 스케줄링 정책은 운영체제마다 다를 수 있다.
- 따라서 정확성이나 성능이 스레드 스케줄러에 따라 달라지는 프로그램은 다른 플랫폼에 이식하기 어렵다.
- 견고한 프로그램을 만들기 위한 가장 좋은 방법은 실행 가능한 스레드의 수가 프로세서 수보다 지나치게 많아지지 않다록 하는 것이다.
    - 실행 준비가 된 스레드는 끝날 때까지 계속 실행되도록 하자.
    - 그러면 스케줄링 정책이 다른 시스템에서도 동작이 크게 달라지지 않는다.
- 스레드는 당장 처리해야 할 작업이 없다면 실행해서는 안 된다.
    - 그래야 실행 가능한 스레드 수가 적게 유지된다.
    - 실행자 프레임워크 예시: 스레드 풀 크기를 적절히 설정하고 작업은 짧게 유지하면 된다.
- 스레드는 절대 바쁜 대기 상태가 되면 안 된다.
    - 스레드 스케줄러의 변덕에 취약하고, 프로세서에 큰 부담을 준다.
    - 예시: `CountDownLatch` 구현
    ```java
    public class SlowCountDownLatch {
        private int count;

        public SlowCountDownLatch(int count) {
            if (count < 0)
                throw new IllegalArgumentException(count + " < 0");
            this.count = count;
        }

        public void await() {
            while (true) {
                synchronized(this) {
                    if (count == 0)
                        return;
                }
            }
        }

        public synchronized void countDown() {
            if (count != 0)
                count--;
        }
    }
    ```
- 특정 스레드가 CPU 시간을 충분히 얻지 못해 프로그램이 간신히 돌아가더라도 `Thread.yield`로 해결하려 하지 말자.
    - 돌릴 때마다 성능이 달라질 수 있다.
    - 테스트할 수단도 없다.
    - 차라리 애플리케이션 구조를 바꿔 동시에 실행 가능한 스레드 수가 적어지도록 하자.
- 스레드 우선순위('Thread.setPriority`)로 해결하려 하지도 말자. 마찬가지의 위험이 따른다.