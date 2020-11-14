## 원자값
> concurrent.atomic 패키지는 lock-free 변수 수정을 위한 클래스를 제공한다.
- 이 클래스들은 다수의 스레드가 같은 인스턴스를 동시에 접근하더라도 올바른 값으로 값을 계산한다.
- 값을 원자적으로 계산하기 위해선 compareAndSet 메서드를 사용해야 한다. (단, 동일한 원자값에 접근하는 스레드가 아주 많을 때는 성능이 떨어진다.)
- 따라서, 높은 경쟁이 예상되는 원자값 을 업데이트할 때는 LongAdder or LongAccumulator 클래스를 사용한다.

- LongAccumulator 는 내부적으로 a1, a2, ... an 변수를 포함하며, 각 변수는 중립요소로 초기화된다.
- 이후 각 변수들이 원자적으로 ai = ai operation v 로 업데이트 되며 이때의 연산은 중위(infix) 누적 연산이다. 
- 이 때의 연산은 결합 법칙과 교환 법칙이 성립해야 한다. 즉, 최종 결과가 중간 값들의 결합하는 순서와 무관해야 한다. ex (1 + 1) +2 == 1 + (1 + 2) 

## ConcurrentHashMap 향상점
> ConcurrentHashMap은 스레드에 안전하게 요소들을 추가하고 삭제할 수 있다.
1) 값 업데이트하기
   - ConcurrentHashMap<String,Long>의 일반적인 put method 로는 thread-safe 하게 연산을 지원하지 않는다. (오잉?)
   - 다른 스레드가 동시에 정확히 같은 카운트를 업데이트하고 있을 수 있기 때문이다.
   - 이를 해결하기 위해, replace 연산을 사용한다. (Atomic 의 compareAndSet 처럼 사용)
   - 아니면 ConcurrentHashMap<String, AtomicLong> 혹은 ConcurrentaHashMap<String, LongAdder> 를 사용할 수 있다.
2) 벌크 연산
   - 벌크 연산은 맵을 순회하며 발견하는 요소들을 대상으로 작업한다.
   - search : null 이 아닌 결과를 return 할 때 까지 각 key/value 에 함수를 적용한다.
   - reduce : 모든 key/value 에 전달받은 함수를 사용해 값을 결합한다.
   - forEach : 함수를 모든 key/value 에 적용한다.
   - 벌크 연산의 종류 : 
        1) operationKeys (키를 대상으로 동작)  
        2) operationValues (값을 대상으로 동작) 
        3) operation (키와 값을 대상으로 동작)
        4) operationEntries : Map.Entry 객체를 대상으로 동작
   - 연산에 parallelism threshold(병렬성 임계값)을 지정하면, 맵이 임계값보다 많은 요소를 가지고 있을 경우 해당 연산이 병렬화 된다.  
3) 집합 뷰

## 병렬 배열 연산
- Arrays.parallelSort 메서드는 기본 타입 또는 객체들의 배열을 정렬할 수 있다.
- Arrays.parallelSetAll 은 요소의 인덱스를 받고 그 위치에 있는 값들을 계산한다.
- Arrays.parallelPrefix 는 각 배열 요소를 주어진 결합 법칙형 여산에 대한 프리픽스의 누적으로 교체한다. (ex 1, 1+2, 1+2+3 ... )

## 완료 가능한 퓨처
> java.util.concurrent 라이브러리는 미래의 어떤 시점에 이용할 수 있는 T 타입 값을 나타내는 Future<T> 인터페이스를 제공한다.
1) 퓨처 합성하기
- ComputableFuture<T> 클래스를 사용하면 thenApply 메서드를 사용해 post-processing function을 전달할 수 있다.
- post-processing function은 blocking 되지 않고, 또다른 future를 return 한다.
2) 합성 파이프라인
- supplyAsync로 ComputableFuture를 생성하여 파이프라인을 시작한다.
- supplyAsync는 Supplier<T> (파라미터가 없고 T를 돌려주는 함수)를 요구한다.
- 이상적으로는 future의 get을 호출하지 않아야 한다.
3) 비동기 연산 합성하기
- ComputableFuture<T> 객체에 액션을 추가하는 방법은 다음과 같다.
  1) thenApply : 결과에 함수를 적용한다.
  2) thenCompose : 결과를 대상으로 함수를 호출하며, 리턴된 퓨처를 실행한다.
  3) handle : 결과 또는 오류를 처리한다.
  4) thenAccept : thenApply와 유사하지만 결과가 void 이다.
  5) whenComplete : handle과 유사하지만 결과과 void 이다.
  6) thenRun : Runnable을 실행하며, 결과가 Void 이다.
- 여러 future를 결합하는 메서드는 다음과 같다.
  1) thenCombine : 둘 모두를 실행하고 주어진 함수로 결과들을 결합한다.
  2) thenAcceptBoth : thenCombine 과 유사하지만 결과가 void 이다.
  3) runAfterBoth : 둘 모두를 완료한 후에 Runnable을 실행한다.
  4) applyToEither : 둘 중 하나에서 결과를 얻을 수 있게 될 때 해당 결과를 주어진 함수에 전달한다.
  5) acceptEither : applyToEither 과 유사하지만 결과가 void 이다.
  6) runAfterEither : 둘 중 하나가 완료한 후에 Runnable을 실행한다.
  7) static allOf : 주어진 모든 ComputableFuture가 완료하면 void 결과로 완료한다.
  8) static anyOf : 주어진 ComputableFuture 중 하나가 완료하면 void 결과로 완료한다.
