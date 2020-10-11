## 반복에서 스트림 연산으로 
컬렉션을 처리할 때 보통은 for문으로 요소를 순회하면서 각 요소별 작업을 처리했다.
```
// for loop
long count = 0;
List<String> example = Arrays.asList("123", "456", "789", "012", "345", "678", "1", "2", "3");
for (String word : example) {
    if (word.length() > 2) count++;
}
```
따라서 위와 같은 '요소 순회' 방식으로는 코드를 병렬화 하기 어려웠다. 

하지만 stream 을 사용하면 다음과 같이 손쉽게 병렬처리가 가능해진다. 
stream을 parallelStream으로 변경하는 것 만으로 스트림 라이브러리가 필터링과 카운팅을 병렬로 수행하게 된다.
```
// stream
count = example.stream().filter(word -> word.length() > 2).count();

// stream with parallel
count = example.parallelStream().filter(word -> word.length() > 2).count();
```
스트림과 컬렉션의 중요한 차이점은 다음과 같다.
1. 스트림은 요소들을 보관하지 않는다. 요소들은 하부읰 컬렉션에 보고나되거나 필요할 때 생성된다.
2. 스트림 연산은 원본을 변경하지 않는다. 대신 결과를 담은 새로운 스트림을 반환한다.
3. 스트림 연산은 가능하면 지연 처리된다.

스트림을 이용해 작업할 때는 연산들의 파이프라인을 세 단게로 설정한다.
1. 스트림을 생성한다. : example.stream()
2. 초기 스트림을 다른 스트림으로 변환하는 중간 연산(intermediate operation)들을 하나 이상의 단계로 지정한다. : example.stream().filter(word -> word.length() > 2)
3. 결과를 산출하기 위해 최종 연산을 적용한다. 이 연산은 앞선 지연 연산(lazy operation)들의 실행을 강제하며, 해당 스트림을 더는 사용할 수 없다. : example.stream().filter(word -> word.length() > 2).count();

## 스트림 생성
> 컬렉션을 스트림으로 바꾸는 것 이외에도 다음과 같이 스트림을 생성할 수 있는 방법들이 있다.

배열이 있다면 정적 Stream.of 메서드를 사용한다.
```
// with array
String[] array = {"123", "456", "789", "012", "345", "678", "1", "2", "3"};
Stream<String> words = Stream.of(array);
```

배열의 일부에서 스트림을 생성도 가능하다.
```
// with array elements
Stream<String> fromToArray = Arrays.stream(array, 0, 1);
```

요소가 없는 스트림은 다음과 같이 생성한다.
```
// empty stream
Stream<String> empty = Stream.empty(); // Stream.<String>empty();
```

무한 스트림을 만들기 위해서는 다음과 같은 두가지 정적 메서드를 사용한다.
```
// infinite stream (generate / no args / Supplier<T>)
Stream<String> infiniteStreamByGenerate = Stream.generate(() -> "Echo");
long count = infiniteStreamByGenerate.count(); // 무한 스트림이기 때문에 count 측정 불가능..?

// infinite stream (iterate / seed value with method / UnaryOperator<T>)
Stream<BigInteger> infiniteStreamByIterate = Stream.iterate(BigInteger.ZERO, n -> n.add(BigInteger.ONE));
count = infiniteStreamByIterate.count();
```

## filter, map, flatMap 메서드
> 스트림 변환은 한 스트림에서 데이터를 읽고, 변환된 데이터를 다른 스트림에 넣는다.

- filter : 메서드 인자로 Predicate<T>를 받는다, 즉 T를 받고 boolean을 리턴한다.
```
// filter
Stream<String> filterStream = example.stream().filter(word -> word.length() >3);
// Stream<T> filter(Predicate<? super T> predicate);
```

- map : 스트림에 있는 값들을 특정 방식으로 변환하고 싶을 때 사용한다. 변환을 수행하는 함수를 파라미터로 전달한다.
```
// map
Stream<String> mapStream = example.stream().map(String::toLowerCase);
// <R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

- flatmap : 여러개의 스트림을 하나의 스트림을 펼쳐서 합칠 때 사용한다.
```
// map
Stream<Stream<Character>> streamOfStream = example.stream().map(word -> characterStream(word));
// flatMap
Stream<Character> flatMapStream = example.stream().flatMap(word -> characterStream(word));
```

## 서브스트림 추출과 스트림 결합
- stream.limit(n) : n 개 요소 이후 끝나는 새로운 스트림을 리턴한다. 이 메서드는 무한 스트림을 필요한 크기로 자를 때 유용하다.
```
// get limited subStream from infinite stream
Stream<Double> limitStream = Stream.generate(Math::random).limit(10);
```

- stream.skip(n) : 처음 n 개 요소를 버린 스트림을 리턴한다.
```
// get skipped subStream from infinite stream
Stream<String> skipStream = Stream.of(array).skip(10);
```

- Stream.concat(..) : 두개의 스트림을 연결 할 수도 있다.
```
// get concat Stream
Stream<Character> concatStream = Stream.concat(characterStream("Hello"), characterStream("World"));
```
- peek : 원본과 동일한 요소들을 포함하는 스트림을 리턴한다. 디버깅을 수행할 때 유용하다.
```
// peek
Stream.iterate(1.0, p -> p*2)
        .peek(e -> System.out.println(e))
        .limit(10);
```

## 상태 유지 변환
- 스트림 변환은 filtering / mapping 등과 같이 결과 스트림을 추출할 때 이전 스트림 요소에 의존하지 않는 무상태 변환(stateless transformation)과 
이전 스트림 요소에 의존적인 상태 유지(stateful transformation)이 존재한다.

예를 들어 distinct 는 중복을 제거한 뒤 원본 스트림으로부터 요소들을 같은 순서대로 돌려주는 스트림을 리턴한다. 이를 상태 유지 (stateful transformation)이라 한다.

```
Stream<String> statefulStream = Stream.of("merrily", "merrily", "merrily", "gently").distinct(); 
```

## 단순 리덕션
- 리덕션(환산)은 최종 연산(terminal operaiton)을 의미하는 것으로, 최종 연산을 적용한 후에는 스트림을 사용할 수 없다.

- 간단한 단순 리덕션에는 min / max / count 등의 메서드가 있다.

- findFirst는 비어 있지 않은 컬렉션에서 첫 번째 값을 리턴한다.
- findAny는 첫 번째 값은 물론 어떤 일치 결과든 괜찮다면 findAny 메서드를 사용한다.
- anyMatch는 단순히 일치하는 요소가 있는지를 알고 싶은 경우에 사용한다. anyMatch는 predicate를 인자로 받으므로 filter를 사용할 필요가 없다.

## 옵션 타입
- Optional<T> 객체는 T 타입 객체 또는 개체가 없는 경우의 레퍼이며, 객체 또는 null 을 가르키는 T 타입 레퍼런스보다 안전한 대안으로 만들어졌다.

1) 옵션 값 다루기
- Optional을 효과적으로 사용하는 핵심은 올바른 값을 소비하거나 대체 값을 생성하는 메서드를 사용하는 것이다.

2) 옵션 값 생성하기
- Optional 객체를 생성하기 위한 정적 메서드에는 Optional.of(result) 또는 Optional.empty()가 있다. 
- Optional.ofNullable(obj)는 obj가 null 이 아니면 Optional.of(obj)를, null 이면 Optional.empty()를 리턴한다.

3) flatMap을 이용해 옵션 값 상수 합성하기
- Optional<T> 를 리턴하는 메서드 f와, 대상 타입 T가 Optional<U>를 리턴하는 메서드 g를 포함하고 있을 경우에 f & g 메서드를 합성하고자하는 경우 
Optional result를 마치 크기가 0 또는 1인 스트림으로 생각해 스트림 of 스트림을 펼쳐내는 방법인 flatMap을 사용해 결과를 추출한다.

## 리덕션 연산
- 합계를 계산하거나 스트림의 요소들을 다른방법으로 결합하고 싶은 경우, reduce 메서드 들 중 하나를 사용할 수 있다.
- 실전에서는 reduce 메서드를 많이 사용하지 않고, 스트림에 맵핑 후 스트림의 합계, 최댓값, 최솟값 계산 메서드를 사용하는 것이 더 쉽다.

## 결과 모으기
- 스트림 작업을 마칠 때 결과를 살펴보기 위해, iterator 메서드를 사용해 반복자를 얻을 수도 있고, toArray를 호출해서 스트림 요소들의 배열을 얻을 수 있다.
- Stream.toArray는 runtime에 제네릭 배열을 생성할 수 없기 때문에 Object[]를 리턴한다. 만약 특정 타입의 배열을 원하는 경우 배열 생성자를 전달한다.

- HashSet으로 결과를 모으려고 하는 경우, HashSet 객체는 thread safe 하지 않으므로 컬렉션을 병렬화 할 경우 단일 HashSet에 직접 넣을 수 없다.
- 따라서, reduce는 사용할 수 없으며, collect를 사용한다.
- collect는 다음과 같은 세가지 인자를 받는다.

1) 공급자 (supplier) : 대상 객체의 새로운 인스턴스를 만든다.
2) 누산자 (accumulator) : 요소를 대상에 추가한다.
3) 결합자 (combiner) : 두 객체를 하나로 병합한다.

- 실전에서는 위와 같은 인자를 제공하는 공통 컬렉터용 팩토리 메서드를 제공하는 Collectors 클래스를 사용하면 된다.

## 맵으로 모으기
- Collectors.toMap 메서드는 각각 맵의 키와 값을 생산하는 두 함수 인자를 받는다.
- 값이 실제 요소여야 하는 경우에는 두 번째 함수로 Function.identity()를 사용한다.
- 키가 같은 요소가 두 개 이상이면 IllegalStateException을 return 하므로, 기존 값과 새값을 받아서 키에 해당하는 값을 결정하는 세 번째 함수 인자를 제공해야 한다.

## 그룹핑과 파티셔닝
- groupingBy 메서드는 그룹 작업을 지원한다.
- 분류함수가 predicate 함수 (즉, boolean 을 리턴하는 함수)라면, partitioningBy를 사용하면 효율적이다. 

- 그룹으로 묶인 요소들의 다운 스트림 처리용으로 몇가지 다른 컬렉터가 제공된다.
1) counting()
2) summing(Int|Long|Double)
3) maxBy, minBy
4) mapping
5) summarizingInt | Double | Long
6) reducing

## 기본 타입 스트림
- 스트림 라이브러리는 기본타입 값들을 직접 저장하는데 특화된 타입을 제공한다.
- IntStream, LongStream, DoubleStream 등이 있다. 
- Stream<Integer> 등의 래퍼 객체로 감싸는 행위는 비효율적이다.
- 기본 타입 스트림 (IntStream / LongStream.. )을 객체 스트림(Stream<Integer>.. )으로 변환하려면 boxed 메서드를 사용한다. 

## 병렬 스트림
- 스트림은 벌크 연산(bulk operation)을 병렬화하기 쉽게 해준다. 
- 스트림 연산들이 병렬로 실행할 때 목적은 연산들을 차례로 실행했을 때와 같은 결과를 리턴하는 것이며, '임의의' 순서로 실행될 수 있다는 점이 중요하다.
- 병렬 스트림 연산에 전달하는 함수가 thread safe 함을 보장하는 것은 개발자의 책임이다.
- 기본적으로 순서유지 컬렉션(배열과 리스트), 범위(range), 발생기(generator) , 반복자 또는 Stream.sorted를 호출해서 얻는 스트림은 순서를 유지한다.
- 몇몇 연산은 순서에 대한 요구 사항을 버리면 더 효과적으로 병렬화 될 수 있다.

## 함수형 인터페이스
- Stream과 Collectors에 속한 메서드들의 파라미터로 나타나는 함수형 인터페이스는 다음과 같다.
1) Supplier<T> 
2) Consumer<T>
3) BiConsumer<T>
4) Predicate<T>
5) ToIntFunction<T> / ToLongFunction<T> / ToDoubleFunction<T>
6) IntFunction<R> / LongFunction<R> / DoubleFunction<R>
7) Function<T, R>
8) BiFunction<T, U, R>
9) UnaryOperator<T>
10) BinaryOperator<T>
