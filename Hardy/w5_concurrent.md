# Concurrent

## ConcurrentHashMap
* thread가 안전한 연산을 할 수 있게 해주는 HashMap
```java
다음은 thread safe 하지 않은 코드
ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
Long oldValue = map.get(word);
Long newValue = oldValue == null ? 1 : oldValue + 1;
map.put(word, newValue);    // 오류 - oldValue를 교체하지 않을 수도 있다.
```
* 값을 안전하게 업데이트 하려면 compute 메소드를 사용해야 한다.<br>
(computeIfPresent, computeIfAbsent, putIfAbsent도 모두 thread safe)
    > map.compute(word, (k, v) -> v == null ? 1 : v + 1);


## Thread
* sleep 재운다
* interrupt 깨운다
* join 기다린다

**개발자가 실 서비스 환경에서 스레드 하나하나 관리하는 것은 불가능. 그래서 Executors 등장**

## Executors
* thread 만들기
* thread 생명주기 관리
* thread로 실행할 작업을 제공할 수 있는 api 제공

**ExecutorService 안에는 ThreadPool이 있고, Blocking Queue가 있다. Thread Pool이 아직 처리하지 못하는 녀석들이 Blocking Queue에 쌓인다.**

### ForkJoin Framework
* 멀티 프로세스를 사용하는 애플리케이션 개발에 유용

## Callable & Future
* Callable -> return 값이 있는 Runnable

## CompletableFuture
* 자바에서 비동기 프로그래밍을 가능케하는 인터페이스
* ForkJoinPool을 사용한다. (Dequeue)

### Future로 하기 어렵던 작업들
* Future를 외부에서 완료 시킬 수 없다. 취소하거나 get()에 타임아웃을 설정할 수는 있었다.
* 블로킹 코드(get())을 사용하지 않고는 작업이 끝났을 때 롤백을 실행할 수 없다.
* 여러 Future를 조합할 수 없다. ex) Event 정보를 가져온 다음 Event에 참석하는 회원 목록 가져오기
* 예외 처리용 api를 제공하지 않는다.
