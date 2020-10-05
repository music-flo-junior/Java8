# 람다를 이용한 프로그래밍
* 람다를 사용하는 주 목적은 적절한 시점까지 코드의 실행을 지연시키기 위함.
* 가능하면 기존 함수형 인터페이스 중 하나를 선택하고, 함수형 인터페이스의 인스턴스를 리턴하는 메소드를 작성하는 것이 좋음.
* 예외를 던지는 람다 표현식을 다룰 때, 무슨 일이 일어나야 하는지 고려해야 한다.
* 제네릭 함수형 인터페이스를 다룰 때 인자 타입에는 ```? super 와일드 카드```를 사용하고, <br>
리턴 타입에는 ```? extends 와일드 카드```를 사용한다.
* 함수를 통해 변환할 수 있는 제네릭 타입을 다룰 때는 map과 flatMap 메소드를 제공하는 방안을 고려한다.

## 지연 실행 - 코드를 나중에 실행하려는 이유
* 별도의 스레드에서 코드 실행
* 코드를 여러 번 실행
* 코드를 적절한 시점에 실행 (ex: 정렬에서의 비교 연산)
* 이벤트 발생 시 코드 실행 (ex: 버튼 클릭)
* 필요할 때만 코드 실행 -> 이벤트 로그 기록

## 람다 표현식의 파라미터
* Runnable과 IntConsumer ?
    * **파라미터가 필요없다면 불필요한 인자를 받도록 강제하지 않는다.**
```java
repeat(10, i -> System.out.println("Countdown: " + (9 - i)));
repeat(10, () -> System.out.println("Countdown: Hardy"));

public static void repeat(int n, IntConsumer action) {
    for (int i = 0; i < n; i++) {
        action.accept(i);
    }
}

public static void repeat(int n, Runnable action) {
    for (int i = 0; i < n; i++) {
        action.run();
    }
}
```

## 함수형 인터페이스 선택
* 문자열 2개를 정수로 맵핑하는 함수를 명시하려면
    * Function<String, String, Integer>
    * (String, String) -> int 또는 Comparator

함수형 인터페이스 | 파라미터 타입 | 리턴 타입 | 추상 메소드명 | 설명 | 다른 메소드
--------------|------------|---------|------------|-----|-----------
Runnable | - | void | run | 인자와 리턴 값 없이 액션 실행
Supplier | - | T | get | T 타입 값을 공급 |
Consumer | T | void | accept | T 타입 값 소비 | chain
BiConsumer<T, U> | void | accept | T와 U 타입 값 소비 | chain
Function<T, R> | T | R | apply | T 타입 인자를 받고 R 타입 리턴 | compose, andThen, identity
BiFunction<T, U, R> | T, U | R | apply | T와 U 타입 인자를 받고 R 타입 리턴 | andThen
UnaryOperator | T | T | apply | T 타입을 대상으로 동작하는 단항 연산자 | compose, andThen, identity
BinaryOperator | T, T | T | apply | T 타입을 대상으로 동작하는 이항 연산자 | andThen
Predicate | T | boolean | test | boolean 값 리턴 | and, or, negate, isEqual
BiPredicate<T, U> | T, U | boolean | test | 인자 2개를 받고 boolean 리턴 | and, or, negate

## 예외 다루기
```java
동기적 연산의 경우
public static void runInSequence(Runnable first, Runnable second, Consumer<Exception> exceptionConsumer) {
    try {
        first.run();
        second.run();
    } catch (Exception e) {
        exceptionConsumer.accept(e);
    }
}
```
```java
비동기 연산의 경우
public static void parallelRunInSequence(Runnable one, Runnable two, Consumer<Exception> exceptionConsumer) {
    BiFunction<Runnable, Consumer<Exception>, Runnable> action = (run, consumer) ->
            () -> {
                try {
                    run.run();
                } catch (Exception e) {
                    consumer.accept(e);
                }
            };

    new Thread(action.apply(one, exceptionConsumer)).start();
    new Thread(action.apply(two, exceptionConsumer)).start();
}
```