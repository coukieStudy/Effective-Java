## 스레드보다는 실행자, 태스크, 스트림을 애용하라

자바의 실행자 프레임워크: ExecutorService ( java.util.concurrent)
```java
// 실행자 생성
ExecutorService exec = Executors.newSingleThreadExecutor();

// 실행자에 실행할 태스크 넘기기
exec.execute(runnable);

// 실행자 종료
exec.shutdown();
```
실행자의 주요 기능
* 특정 태스크가 완료되기를 기다린다(.get())
* 태스크 모음 중 아무것(invokeAny) 혹은 모든 태스크(invokeAll)가 완료되기를 기다린다.
* 실행자 서비스가 종료하기를 기다린다(awaitTermination 메서드)
* 완료된 태스크들의 결과를 차례로 받는다(ExecutorCompletionService 이용)
* 태스크를 특정 시간에, 혹은 주기적으로 실행하게 한다(ScheduledThreadPoolExecutor 이용)

원하는 실행자 대부분은 java.util.concurrent.Executors의 정적 팩터리들을 이용해 생성할 수 있다.
특수한 목적의 실행자는 ThreadPoolExecutor 클래스를 직접 이용하면 된다.

작은 프로그램이라면 Executors.newCachedThreadPool 을 사용하면 좋지만, 위의 실행자에서는 요청받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임되어 실행되며 가용한 스레드가 없다면 새로 생성한다. 따라서 무거운 프로덕션 서버에서는 스레드 개수를 고정한 Executors.newFixedThreadPool을 선택하는 것이 낫다. 

작업 큐를 가급적 직접 만들지 말고, 스레드를 직접 다루는 일도 가급적 하지 말아야 한다. 실행자 프레임워크에서는 작업 단위(태스크)와 실행 메커니즘이 분리된다.

태스크에는 Runnable과 Callable이 있으며, Callable은 값을 반환하고 임의의 예외를 던질 수 있다.

자바7이 되면서, 실행자 프레임워크는 fork-join 태스크를 지원하도록 확장되었다. 포크-조인 태스크, 즉 ForkJoinTask의 인스턴스는 작은 하위 태스크로 나뉠 수 있고, ForkJoinPool을 구성하는 스레드들이 이 태스크들을 처리하며, 일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 가져와 대신 처리할 수도 있다. 이렇게 하여 모든 스레드가 바쁘게 움직여 CPU를 최대한 활용하면서 높은 처리량과 낮은 지연시간을 달성한다. 
