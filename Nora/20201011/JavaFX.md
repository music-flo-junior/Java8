* 핵심내용
- 람다 표현식을 사용하는 주 이유는 적절한 시점까지 코드와 실행을 지연하기 위해서다.
- 람다 표현식을 실행할 때 필요한 모든 데이터를 입력으로 제공해야한다.
- 가능하면 기존 함수형 인터페이스 중 하나를 선택한다.
- 종종 함수형 인터페이스의 인스턴스를 리턴하는 메서드를 작성하는 일이 유용하다.
- 변환들을 다룰 때는 각 변환을 어떻게 합성할지 고려한다.
- 변환들을 지연해서 합성하려면, 모든 지연 변환의 목록을 관리하고 마지막에는 변환들을 적용해야 한다.
- 람다를 여러 번 적용해야 하는 경우, 종종 헤딩 직업을 동시에 실행하는 하위 작업들로 분리할 기회가 있다.
- 예외를 던지는 람다 표현식을 다룰 때는 무슨 일이 일어나야 하는지 생각해야 한다.
- 제네릭 함수형 인터페이스를 다룰 때 인자 타입에는 ? super 와일드카드를 사용하고, 리턴 타입에는 ? extends 와일드카드를 사용한다.
- 함수를 통해 변환할 수 있는 제네릭 타입을 다룰 때는 map과 flatMap 메서드를 제공하는 방안을 고려해본다.

* 01 지연 실행
- 모든 람다의 핵심은 지연 실행(deferred execution)이다.
- 다음처럼 코드를 나중에 실행하는 이유는 많다.
1. 별도의 스레드에서 코드 실행
2. 코드를 여러 번 실행
3. 알고리즘에서 코드를 적절한 시점에 실행(예를 들면, 정렬에서 비교 연산)
4. 어떤 일이 발생했을 때 코드 실행(버튼 클릭 시, 데이터 도착 시 등
5. 필요할 떄만 코드 실행

- 필요할 때만 코드를 실행하는 일이 바로 람다의 용도다. 
- 표준 형식은 코드를 인자 없는 람다로 감싸는 것이다.
() -> "x:"+x+",y:"+y

- 이제 다음과 같이 동작하는 메서드를 작성해야한다.
1. 람다를 받는다.
2. 람다를 호출해야 하는지 검사한다.
3. 필요할 때 람다를 호출한다.

- 람다를 받으려면 함수형 인터페이스를 골라야 한다.(또는 드문 경우 직접 제공한다)
public static void info(Logger logger, Supplier<String> message){
    if(logger.isLoggable(Level.INFO)){
        logger.info(message.get());
    }
} 
Logger 클래스의 isLoggsble 메서드를 사용해 INFO 메시지를 로그로 남겨야하는지 판단한다.
로그로 남겨야 한다면 추상메서드 get을 호출하여 람다를 실행한다.

- 메시지 로깅을 지연하는 일은 자바8 라이브러리 설계자들이 깨우쳐준 멋진 생각이다.
다른 로깅 메서드는 물론 info 메서드는 이제 Supplier<String>을 받는 변종이 존재한다.
따라서 logger.info(()->"x:"+x+",y:"+y)를 직접 호출할 수 있다.

* 02 람다 표현식의 파라미터
- 예제 1
사용자에게 비교자를 제공하도록 요구하는 경우에는 해당 비교자에서 인자 두 개를 받는다.
Arrays.sort(names,(s,t) -> Integer.compare(s.length(),t.length()));

- 예제2
다음 메서드는 액션을 여러번 반복한다.
public static void repeat(int n, IntConsumer action){
    for(int i = 0; i < n; i++){
        action.accept();
    }  
}
IntConsumer 클래스의 객체를 사용하는 이유는 몇번 째 반복에서 실행하고 있는지를 액션에 알려주는데,
이는 유용한 정보가 될 수 있다. 액션은 파라미터로 들어온 입력을 캡쳐해야 한다.
repeat(10,i -> System.out.println("CountDown : "+ (9-i));

- 예제3
이벤트 처리기 예제이다.
button.setOnAction(event -> 액션);
event 객체는 액션에서 필요할 만한 정보를 실어 나른다.

- 일반적으로 필요한 모든 정보를 인자로 전달할 수 있도록 알고리즘을 설계한다.
하지만 인자가 거의 필요 없다면 사용자에게 불필요한 인자들을 받도록 강제하지 않는 방안도 있다.
1. 인자가 필요한 경우
public static void repeat(int n, Runnable action){
    for(int i = 0; i< n; i++) action.run(); 
}
2. 인자가 필요없는 경우
repeat(10,() -> System.out.println("Hello!world"));

* 03 함수형 인터페이스 선택
- 대부분 함수형 프로그래밍 언어에서 함수 타입은 구조적이다.
- 문자열 두 개를 정수로 맵핑하는 함수를 명시하려면 Function2<String, String, Integer> 
또는 (String,String) -> int 형태의 타입을 사용한다.
자바에서는 이 대신 Comparator<String> 같은 함수형 인터페이스를 사용해 함수의 의도를 선언한다. 
프로그래밍 언어 이론에서는 명목적 타이핑이라고 한다.

- 특정의미가 없는 모든 함수를 받고자 하는 상황이 많다. 이 용도로 사용할 수 있는 여러 가지 레네릭 함수 타입이 있다.

종류	        추상 메소드 특징	                                        비고
Function	주로 매개값을 연산하고 결과를 리턴	                            BiFunction
Consumer	인자는 있고, 리턴값은 없음	                                BiConsumer
Supplier	인자는 없고, 리턴값은 있음	 
Operator	인자도 있고, 리턴값도 있음, 주로 매개값을 연산하고 결과를 리턴	 
Predicate	인자는 있고, 리턴값은 boolean, 매개값을 조사하고 true/false를 리턴	BiPredicate

- 특정 기준을 만족하는 파일을 처리하는 메서드를 작성할 때에는 Predicate의 사용을 강력히 추천한다.

* 04 함수리턴
- 리턴타입이 함수형 인터페이스인 메서드가 존재한다.
- Image brightenedImage = transform(image,brighten(1.2));
brighten 메서드는 함수를 리턴한다. 

* 05 합성
- public static <T> UnaryOperator<T> compose(UnaryOperator<T> op1, UnaryOperator<T> op2){
    return t -> op2.apply(op1.apply(t));
}

이제 다음과 같이 호출할 수 있다.

Image finalImage = transform(image,compose(Color::brighter,Color::grayscale));

- 일반적으로 사용자가 차례로 효과를 적용할 수 있는 라이브러리를 구축할 때, 라이브러리 사용자가 해당 효과들을 합성할 수 있게
해주는것이 좋다.                                                       
                                                                                                                                                                                                   