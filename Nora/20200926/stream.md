* 핵심내용
- 반복자(lterator)는 특정 순회 전략을 내포하므로 효율적인 동시 실행을 방해한다.
- 컬렉션, 배열, 발생기, 반복자로부터 스트림을 생성할 수 있다.
- 요소를 선택하는데 filter를 사용하고, 요소를 변환하는데 map을 사용한다.
- 스트림을 변환하는 다른 연산으로는 limit, distinict, sorted가 있다.
- 스트림에서 결과를 얻으려면 count, max, min, finFirst 또는 findAny 같은 리덕션 연산자를 사용한다.
이들 메서드 중 몇몇은 Optional 값을 리턴한다.
- Optional 타입은 null 값을 다루는 안전한 대안을 목적으로 만들어졌다. Optional 타입을 안전하게 사용하려면 
ifPresent와 orElse메서드를 이용한다.
- 스트림 결과들을 컬렉션, 배열, 문자열 또는 맵으로 모을 수 있다.
- Collectors 클래스의 groupingBy와 PartitioningBy 메서드는 스트림의 내용을 그룹으로 분할하고 각 그룹
의 겨로가를 얻을 수 있게 해준다.
- 기본 타입인 int, long, double용으로 특화된 스트림이 있다.
- 병렬 스트림을 이용할 때는 부가 작용(side effect)을 반드시 피해야 하고, 순서 제약을 포기하는 방안도 고려한다.
- 스트림 라이브러리를 사용하려면 몇 가지 함수형 인터페이스와 친숙해져야 한다.

* 01 반복에서 스트림 연산으로
- 컬렉션을 처리할 때, 보통은 요소들을 순회하면서 각 요소를 대상으로 작업한다.

- 자바8의 벑크 연산(bulk operation) 
> long count = words.stream().filter(w->w.length() > 12).count();
> stream 메서드는 words 리스트의 스트림을 돌려준다.
> filter 메서드는 12글자보다 긴 단어만 담은 다른 스트림을 리턴한다.
> count 메서드는 이 스트림을 결과로 리듀스한다.

- 스트림은 데이터를 변환하고 추출할 수 있게 해주어 겉으로는 컬렉션과 유사하게 보인다. 하지만 중대한 차이점이 있다.
1. 스트림은 요소들을 보관하지 않는다. 요소들은 하부의 컬렉션에 보관되거나 필요할 때 생성된다.
2. 스트림 연산은 원본을 변경하지 않는다. 대신 결과를 담은 새로운 스트림을 반환한다.
3. 스트림 연산은 가능하면 지연 처리된다. 지연 처리란 결과가 필요하기 전에는 실행되지 않음을 의미한다.
예를 들어, 긴 단어를 모두 세는 대신 처음 5개 긴 단어를 요청하면, filter 메서드는 5번 째 일치 후 필터링을 중단한다.
결과적으로 심지어 무한 스트림도 만들 수 있다.

- 스트림은 쉽게 병렬화할 수 있다.
> long count = words.parallelStream().filter(w->w.length() > 12).count();
> 단순히 stream을 parallelStream으로 변경하는 것만으로 스트림 라이브러리가 필터링과 카운팅을 병렬로 수행하게 할 수 있다.

- 스트림은 '어떻게가 아니라 무엇을'원칙을 따른다. 따라서 스트림은 무엇을 어떻게 해야하는지 정의한다.

- 스트림을 이용해 작업할 때는 연산들의 파이프라인을 세단계로 설정한다.
1. 스트림을 생성한다.
2. 초기 스트림을 다른 스트림으로 변환하는 중간연산들을 하나 이상의 단계로 지정한다.
3. 결과를 산출하기 위해 최종연산을 적용한다. 이 연산은 앞선 지연 연산들의 실행을 강제한다. 이후로는 해당 스트림을 더는 사용할 수 없다.
> 앞선 예제에서는 stream 또는 parallelStream 메서드로 스트림을 생성한다. filter 메서드로 스트림을 변환하고, count는 최종 연산이다.

- 스트림 연산들은 요소를 대상으로 실행할 때 스트림에서 호출된 순서로 실행되지 않는다. 앞선 예제에서는 count가 호출되기 전에는 
아무 일도 일어나지 않는다. count 메서드가 첫 번째 요소를 요청하면, filter메서드가 길이 > 12인 요소를 찾을 때 까지 요소들을 요청하기 시작한다.

* 02 스트림 생성
- 자바 8에서 Collection 인터페이스에 추가된 stream 메서드를 이용해 컬렉션을 스트림으로 바꿀 수 있다.
> Stream<String> words = Stream.of(contents.split("[\\P{L}]+""));

- of 메서드는 가변 인자 파라미터를 받기 때문에 인자 개수가 몇 개든 스트림을 생성할 수 있다.
> Stream<String> song = Stream.of("gently","down","the","stream");

- 배열의 일부에서 스트림을 생성하려면 Arrays.stream(array, from, to)를 사용한다.
요소가 없는 스트림을 생성하려면 정적 Stream.empty 메서드를 사용한다.

- Stream 인터페이스는 무한 스트림을 만드는 두가지 정적 메서드를 포함한다.
1. generate 메서드는 인자 없는 함수를 받는다. 스트림 값이 필요할 떄는 이 함수를 호출해서 값을 생성한다.
> Stream<String> echos = Stream.generate(()->"Echo");
> Stream<Double> randoms = Stream.generate(Math::random);
2. 무한 수열을 만들어내려면 iterate 메서드를 사용한다. iterate 메서드는 시드값과 함수를 받고, 해당 함수를 이전 결과에 반복적으로 적용한다.
> Stream<BigInteger> integers = Stream.iterate(BigInteger.ZERO, n -> n.add(BigInteger.ONE));

* 03 filter,map,flatMap 메서드
- filter : 특정 조건과 일치하는 모든 요소를 담은 새로운 스트림으로 변환한다.
- filter의 인자는 Predicate<T>, 즉 T를 받고 boolean을 리턴하는 함수다.
- map : 스트림에 있는 값들을 특정 방식으로 변환하고 싶을 때, 사용하는 메서드. 변환을 수행하는 함수를 파라미터로 전달한다.
> Stream<String> lowercaseWords = words.map(String::toLowerCase);
- flapMap : 스트림을 문자들의 스트림으로 펼쳐내려면 map 대신 flapMap을 사용.
- 제네릭 타입 G, 타입 T를 G<U>로 변환하는 함수 f 그리고 타입 U를 G<V>로 변환하는 함수 g가 있다.
그러면 flatMap을 사용하여 이 함수들을 함성할 수 있다.(모나드이론에서 핵심개념)
> stream<Character> letters = words.flatMap(w->characterStream(w))

* 04 서브스트림 추출과 스트림 결합
- stream.limit(n) : n개 요소 이후 끝나는 대로 새로운 스트림을 리턴한다.
- 원본 스트림이 n보다 짧은 경우는 원본 스트림이 끝날 때 까지 
- 무한 스트림을 필요한 크기로 잘라낼 때 특히 유용하다
> Stream<Double> randoms = Stream.generate(Math::random).limit(100);

- stream.skip(n) : 처음 n개 요소를 버린다.
> Stream<String> words = Stream.of(contents.split("[\\P{L}]+")).skip(1);
> split 메서드 동작방식으로 인해 첫 번째 요소가 불필요한 빈 문자열이다. 따라서 skip을 호출하면 불필요한 요소가 제거 가능하다.

- Stream 클래스의 정적 concat 메서드를 이용하면 두 스트림을 연결할 수 있다.
> Stream<Character> combined = Stream.concat(characterStream("Hello"),characterStream("World"));
> 물론 첫번째 스트림이 무한 스트림이면 안된다 -> 두번째 스트림이 연결될 기회를 얻지 못한다.

- Stream 클래스의 peek 메서드는 원본과 동일한 요소들을 포함하는 다른 스트림을 돌려주지만, 전달받은 함수를 요소 추출 시 마다 호출한다.
- 디버깅을 수행할 때 매우 유용하다.
> Object[] powers = Stream.iterate(1.0,p->p*2)
    .peek(e->System.out.println("Fetching"+e))
    .limit(20).toArray();

* 05 상태 유지 변환
- 무상태 변환 : 필터링 또는 맵핑된 스트림에서 요소를 추출할 때 결과가 이전 요소에 의존하지 않는다.
- 상태 유지 변환 
> distinct 메서드 : 중복을 제거하는 점을 제외하면 원본 스트림으로 부터 요소들을 같은 순서로 돌려주는 스트림을 리턴
이 경우에는 이미 만난 요소들을 확실히 기억해야한다.

- sorted 메서드는 여러 버전이 있다. 
1. Comparable 요소들의 스트림을 대상으로 작업
2. Comparator을 받는 버전
> Stream<String> longestFirst = words.sorted(Comparator.comparing(String::length).resersed());
- 물론 스트림을 사용하지 않고도 컬렉션을 정렬할 수 있으나, 정렬과정이 스트림 파이프라인의 일부일 때 유용하다.
- Collections.sort 메서드는 컬렉션을 직접 정렬하나, Stream.sort 메서드는 새롭게 정렬된 스트림을 리턴한다.

* 06 단순 리덕션
- 리덕션 메서드는 스트림을 프로그램에서 사용할 수 있는 값으로 리듀스 한다.
- 리덕션은 최종연산이다. 최종 연산을 적용한 후에는 스트림을 사용할 수 없다.

- count 메서드 / max,min 메서드 
> 해당 메서드들은 스트림이 떄로는 비어있을 수 있기 떄문에 Optional<T> 값을 리턴한다.
> Optional<String> largest = words.max(String::compareToIgnoreCase);
> if(largest.isPresent()) System.out.println("largest :: " + largest.get());

- findFirst 메서드 : 비어있지 않은 컬렉션에서 첫번째 값을 리턴
- findAny 메서드 : 첫번째 값은 물론 어떤 일치 결과든 괜찮은 경우 사용 (병렬화할 때 유용)
- anyMatch 메서드 : 단순히 일치하는 요소가 있는지 알고싶은 경우에 사용
- allMatch / noneMatch 메서드 : 모든 요소가 프레디케이트와 일치하거나 아무 것도 일치하지 않을 때 사용

* 07 옵션 타입
- Optional<T> 객체는 T 타입 객체 또는 객체가 없는 경우의 래퍼다.
- 객체 또는 null을 가리키는 T 타입 레퍼런스보다 안전한 대안으로 만들어졌다.

- get() : 감싸고 있는 요소가 존재할 떈 요소를 얻고, 그렇지 않으면 NoSuchElementException을 던진다.
> Optional<T> optionalValue = optionalValue.get().someMethod();

- isPresent() : Optional<T> 객체가 값을 포함하는지 알려준다.
> if(optionalValue.isPresent()) optionalValue.get().someMethod();                     

- 옵션 값 다루기

- 옵션 값 생성하기

- flatMap을 이용해 옵션 값 함수 합성하기
                                                                                                                                                                                                               