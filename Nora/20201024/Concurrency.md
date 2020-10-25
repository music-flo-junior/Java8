* 병행성 향상점
- 병행 프로그래밍은 어렵고, 적당한 도구 없이는 더욱 어렵다.
- 초기 자바 릴리스는 병행성을 최소로 지원했기 때문에, 프로그래머들은 교착 상태와 경쟁 조건을 포함하는 코드를 부지런히 만들어냈다.
- 자바 5 이전에는 견고한 java.util.concurrent 패키지가 나오지 않았다.
- 이 패키지는 스레드에 안전한 컬렉션과 스레드 풀을 제공해 많은 애플리케이션 프로그래머가 잠금을 사용하거나 스레드를 시작하지 않고도 병행 프로그램을 작성할 수 있게 해준다.

* 핵심내용
- updateAndGet/accumulateAndGet 메서드 덕분에 원자 변수 업데이트가 간단해졌다.
- 높은 경쟁 상황에서는 LongAccumulator/DoubleAccumulator가 AtomicLong/AtomicDouble 보다 효율적이다.
- compute와 merge 메서드 덕분에 ConcurrentHashMap에 있는 엔트리의 업데이트가 간단해졌다.
- ConcurrentHashMap은 이제 search, reduce, forEach 같은 벌크 연산을 지원하며, 각각 키, 값, 키와 값, 엔트리에 적용하는 변종을 제공한다.
- 집합 뷰는 ConcurrentHashMap을 Set으로 사용할 수 있게 해준다.
- Arrays 클래스는 병렬정렬, 채우기, 프리픽스 연산을 위한 메서드를 포함한다.
- 완료 가능한 퓨처(completable future)는 비동기 연산들을 합성할 수 있게 해준다.

* 01 원자값
- 자바 5 이후로 java.util.concurrent.atomic 패키지는 잠금없는 변수 수정을 위한 클래스를 제공한다.
- public static AtomicLong nextNumber = new AtomicLong();
long id = nextNumber.incrementAndGet();
- incrementAndGet 메서드는 원자적으로 AtomicLong을 증가시키고, 증가한 값을 리턴한다.
다시 말해, 값을 얻어서 여기에 1을 더해 설정한 후 새로운 값을 주는 과정에서 방해를 받지 않는다.
이때 다수의 스레드가 같은 인스턴스를 동시에 접근하더라도 올바른 값이 계산되어 리턴됨을 보장받는다.
- 값을 원자적으로 설정하고, 더하고, 빼는 메서드가 있지만 좀 더 복잡한 업데이트를 수행하려면 compareAndSet 메서드를 사용해야 한다.
- compareAndSet 메서드는 잠금을 사용하는 방법보다 빠른 프로세서 연산과 연관된다.
- 자바 8에서는 더는 상투적으로 사용해온 루프를 작성할 필요가 없다. 대신 변수를 업데이트하는 람다 표현식을 제공하면 해당 업데이트가 적용된다.
largest.updateAndGet(x -> Math.max(x,osberved));
또는 다음과 같이 호출할 수 있다.
largest.accumulateAndGet(observed,Math::max);
- accumulateAndGet 메서드는 원잣값과 제공받은 인자를 결합하는 이항 연산자를 받는다.
- 기존 값을 리턴하는 getAndUpdate와 getAndAccumulate도 있다.

- 동일한 원잣값을 접근하는 스레드가 아주 많을 때는 지나친 업데이트로 너무 많은 재시도가 필요하기 때문에 성능이 떨어진다.
- 자바 8에서는 이 문제를 해결하려고 LongAdder와 LongAccumulator 클래스를 제공한다.
LongAdder는 각각을 모두 합하면 현재 값이 되는 여러 변수로 구성된다. 여러 스레드가 서로 다른 서맨드를 업데이트할 수 있고,
스레드 수가 증가하면 새로운 서맨드가 자동으로 제공된다. 이 방법은 모든 작업을 마치기 전에는 합계 값이 필요하지 않은 일반적인 상황에서 효율적이다.
따라서 이로 인한 성능 향상이 상당할 수 있다.
- 높은 경쟁을 예상할 때는 AtomicLong 대신 무조건 LongAdder를 사용해야 한다.
- LongAdder의 메서드 이름들은 AtomicLong과 약간씩 다르다. 카운터를 증가시키려면 increment를, 수량 추가할 때는 add를, 합계를 추출할 때는 sum을 호출한다.
- LongAccumulator는 이 개념을 임의의 누적 연산으로 일반화한다. LongAccumulator의 생성자에는 필요한 연산과 해당 연산의 중립요소를 제공한다.
새로운 값을 집어넣으려면 accumulate를 호출하고, 현재 값을 얻으려면 get을 호출한다.

- 자바 8은 낙관적 읽기를 구현하는데 사용할 수 있는 StampedLock 클래스도 추가됐다.
애플리케이션 프로그래머가 잠금을 사용하는 것은 권장하지 않지만, StampedLock을 이용한 잠금 사용법이 있다.
먼저 tryOptimisticRead을 호출하여 스탬프를 얻는다. 다음으로 값을 읽고 스탬프가 여전히 유효한지 검사한다.
스탬프가 아직 유효하면 값을 이용할 수 있다. 그렇지 않으면 읽기 잠금을 얻는다.

* 02 ConcurrentHashMap 향상점
- 자바 5 이후로 ConcurrentHashMap이 병행 프로그래밍의 주력 자료 구조가 되었다.
당연히 ConcurrentHashMap은 스레드에 안전하다. 더구나 여러 스레드가 서로 블록하지 않고도 테이블의 서로 다른 부분을 동시에 업데이트할 수 있게 해주므로 상당히 효율적이다.

- 값 업데이트 하기
- replace을 사용해 기존 값을 새로운 값으로 교체한다.
- ConcurrentHashMap<String, AtomicLong>을 사용하거나 자바 8에서는 ConcurrentHashMap<String, LongAddr>를 사용할 수 있다.
- ConcurrentHashMap에 null 값을 저장할 수 없다. 주어진 키가 맵에 존재하지 않음을 나타내는 데 null 값을 사용하는 다양한 메서드가 있기 때문이다.
- 예제
map.pulIfAbsent(word, new LongAdder());
map.get(word).increment();
첫번째 문장은 원자적으로 증가 시킬 수 있는 LongAdder가 존재함을 보장한다.
putIfAbsent는 맵핑된 값을 리턴하므로, 두 문장을 합칠 수 있다.
map.putIfAbsent(word, new LongAdder()).increment();

- 자바 8은 원자적 업데이트를 더 편리하게 해주는 메서드를 제공한다.
compute 메서드는 키와 새로운 값을 계산하는 함수를 전달하여 호출한다.
map.compute(word, (k,y) -> v == null ? 1 : v+1);
- 각각 기존 값이 있을 때와 기존 값이 없을 때만 새로운 값을 계산하는 computeIfPresent, computeIfAbsent 변종도 있다.

- 키가 처음으로 추가될 때 뭔가 특별한 작업을 해야 할 때도 있다. merge 메서드를 사용하면 이 경우에 특히 편리하다.
이 메서드는 키가 아직 존재하지 않을 때 사용할 초기값을 파라미터로 받는다. 키가 존재하면 파라미터로 전달한 함수가 기존 값과 초기값을 받으며 호출된다.
map.merge(word, 1L, (existingValue, newValue) -> existingValue + newValue);

- compute나 merge에 전달한 함수가 null을 리턴하면 기존 엔트리가 맵에서 제거된다.

- 벌크연산
- 자바 8은 서로 다른 스레드들이 같은 맵을 대상으로 작업하는 경우에도 안전하게 실행할 수 있는 벌크 연산을 병행 해시 맵에 제공한다.
벌크연산은 맵을 순화하며 발견하는 요소들을 대상으로 작업한다. 제때 맵의 스냅샷을 동결하려는 수고는 하지 않는다.
벌크 연산이 실행하는 동안에 맵이 수정되지 않음을 알고 있지 않는 한 결과를 맵의 상태에 대한 근사치로 취급해야 한다. 
- 연산종류
1. search : 함수가 널이 아닌 결과를 돌려줄 때까지 각 키 그리고 값에 함수를 적용한다. 그런 다음 검색을 종료하고 함수의 결과를 리턴한다.
2. reduce : 제공받은 누적 함수를 이용해 모든 키 그리고 값을 결합한다.
3. forEach : 함수를 모든 키 그리고 값에 적용한다.
- 각 연산 네가지 버전
1. operationKeys : 키를 대상으로 동작
2. operationValues : 값을 대상으로 동작
3. operation : 키와 값을 대상으로 동작
4. operationEntries : Map.Entry 객채를 대성으로 동작

- 집합 뷰 
- 정적 newKeySet 메서드는 실제로는 ConcurrentHashMap<K, Boolean>을 감싸고 있는 set<k>를 돌려준다.

* 03 병렬 배열 연산
- Arrays 클래스는 다수의 병렬 연산을 제공한다. 정적 Arrays.parallelSort 메서드는 기본 타입 또는 객체들의 배열을 정렬할 수 있다.
- paralleSetAll 메서드는 전달받은 함수에서 계산한 값들로 배열을 채운다. 해당 연산은 병렬화의 이점을 얻는다.

* 04 완료 가능한 퓨처
- 퓨처 
