# Item 80. 스레드보다는 실행자, 태스크, 스트림을 애용하라

## Executor Framework(실행자 프레임워크)

- 자바5(1.5)부터 추가되었으며 `java.util.concurrent` 패키지내에 존재

### Executor Framework 가 하는 일

1. Thread 생성 : thread를 생성하거나 thread pool을 만드는 method 를 제공
2. Thread 관리 : thread 의 생명주기를 관리
3. Task 제출 및 실행

```java
// ExecutorService instace 생성
ExecutorService executor = Executors.newSingleThreadExecutor();

// Task 제출 및 실행
executor.execute(runnable);

// graceful shutdown(유예를 가진 종료, 실패 시 VM이 종료되지 않는다)
executor.shutdown();
```

### 실행자 서비스의 주요 기능

- 특정 태스크가 완료되기를 기다린다
- 태스크 모음중 하나(invokeAny) 또는 모든 태스크(invokeAll)가 완료되기를 기다린다
- 실행자 서비스가 종료하기를 기다린다(awaitTermination)
- 완료된 태스크들의 결과를 차례로 받는다(ExecutorCompletionService 이용)
- 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다(ScheduledThreadPoolExecutor 이용)

### 실행자 서비스 선택

- CachedThreadPool
  - 요청받은 태스크들이 즉시 스레드에 위임되어 실행(큐에 쌓이지 않는다)
  - 작은 프로그램 또는 가벼운 서버에 적합 (Executors.newCachedThreadPool)
- Executors.newFixedThreadPool
  - 스레드 개수를 고정
  - 무거운 프로덕션 서버에 적합
- ThreadPoolExecutor
  - 스레드 풀의 모든 속성 설정 가능

<br>

## Executor Framework 를 사용해야 하는 이유

스레드를 직접 다루면 Thread가 작업 단위와 수행 메커니즘 역할을 모두 수행하게 된다.
반면 실행자 프레임워크에서는 작업 단위와 실행 메커니즘이 분리된다.

### 작업 단위

> 태스크 : 작업 단위를 나타내는 핵심 추상 개념

- Runnable
- Callable
  - 값을 반환하고 임의의 예외를 던질 수 있다

### 실행 메커니즘

> 실행자 서비스 : 태스크를 수행하는 일반적인 매커니즘

<br>

## ForkJoinTask

- 자바7부터 지원

- 포크-조인 태스크는 ***포크-조인 풀(ForkJoinPool)*** 이라는 실행자 서비스가 실행

- ForkJoinTask의 인스턴스는 작은 하위 태스크로 나뉠 수 있고, ForkJoinPool 을 구성하는 스레드들이 이 태스크들을 처리

  일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 가져와 대신 처리할 수도 있다

- CPU를 최대한 활용하면서 높은 처리량과 낮은 지연시간을 달성한다

- 포크-조인 풀을 이용해 만든 병렬 스트림([아이템 48](Item48.md))을 이용하면 적은 노력으로 그 이점을 얻을 수 있다

<br>

## 참고

- [Executor Framework](https://engkimbs.tistory.com/589)

- [오류를 잡자 : TCP에는 우아한 종료라는 것은 없다.](https://sunyzero.tistory.com/269)



