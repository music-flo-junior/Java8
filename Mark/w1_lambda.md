## 왜 람다인가?
> lambda란 java8에 추가된 functional programming을 지원하는 기술이다.

기존 OOP 방식의 프로그래밍에선 모든 기능들을 object 기반으로 작성했다. 따라서 모든 code blocks 들은 class & object 에 연관되어 있었다.
따라서 기존 자바 개발자들은 단순한 **'기능'**을 작성하는데도 이를 함수로 가지고 있는 class를 작성하고, 이를 인스턴스로 생성해 해당 인스턴스가 해당 **'기능'**을 수행하도록 명령해야 했다.

예를 들어 문자열 정렬을 custom comparator를 이용해 문자열을 정렬하고자 한다면 다음과 같이 Comparator의 compare를 구현한 객체를 생성해 Arrays.sort 메서드에 전달해야 했다.
```
class LengthComparator implements Comparator<String> {
   public int compare(String first, String second) {
        return Integer.compare(first.length(), second.length());
   }
}

Arrays.sort(strings, new LengthComparator());
```

## 람다 표현식 문법
> 람다를 사용하면 단순한 코드 블럭을 전달해 **기능**을 수행할 수 있다.

위에서 LengthComparator로 구현한 문자열 정렬은 람다를 사용하면 다음과 같이 작성할 수 있다.
이때 람다 표현식의 결과 타입은 따로 지정하지 않으며, 결과타입은 컴파일러에 의해 문맥으로부터 추정된다.
```
(String first, String second) -> Integer.compare(first.length(), second.length());
```

만약, 코드에서 표현식 하나로는 표현할 수 없는 계산을 수행한다면, 중괄호 {}로 감싸고 명시적인 return 문을 사용해 작성한다.
```
(String first, String second) -> {
  if ( first.length() < second.length()) return -1;
  else if (first.length() > seconde.length()) return 1;
  else return 0;
}
```

컴파일러가 람다표현식의 파라미터 타입을 추정할 수 있는 경우 타입을 생략할 수 있다.
```
(first, second) -> Integer.compare(first.length(), second.length())
```

람다 표현식이 파라미터를 받지 않는다면, 파라미터 없는 메서드와 마찬가지로 빈 괄호를 사용한다.
```
() -> { for (int i=0; i< 1000; i++) doWork(); }
```

타입 한개를 파라미터로 받는 경우 괄호를 생략할 수 있다.
```
event -> System.out.println("Thanks for clicking!");
```

## 함수형 인터페이스
> 단일 추상 메서드(single abstract method)를 갖춘 인터페이스의 객체를 기대할 때 람다 표현식을 사용할 수 있으며, 이를 함수형 인테퍼이스(functional interface)라고 한다.

앞서 살펴본 Arrays.sort 메서드는 두 번째 파라미터로 단일 메서드를 갖춘 인터페이스인 Comparator의 구현체를 요구한다. 따라서 우리는 구현체가 아닌 람다로 작성된 코드블럭을 전달할 수 있다.
```
Arrays.sort(words, (first, second) -> Integer.compare(first.length(), second.length()));
```

림다도 결국에는 인터페이스의 메서드를 구현하는 것이지만, 내부적으로 interface implementation은 인스턴스 객체를 생성해 작업을 수행하고,
람다는 인터페이스의 메서드를 calling 하는 방식으로 동작한다.

## 메서드 레퍼런스
> 코드블럭을 통해 전달하고자 하는 메서드가 이미 존재하는 경우, 메서드 레퍼런스를 사용해 조금더 간결하게 람다식을 작성할 수 있다.

예를 들어 기존에 다음과 같이 버튼을 클릭할 때마다 이벤트 객체를 출력하는 람다식이 있는 경우
```
button.setOnAction(event -> system.out.println(event));
```

메서드 레퍼런스를 사용하면 다음과 같이 간결하게 작성할 수 있다.
```
button.setOnAction(system.out::println);
```

메서드 레퍼런스를 사용할 수 있는 주요 세가지 경우는 다음과 같다.
- objcet::instanceMethod
- Class::staticMethod
- Class::instanceMethod

## 생성자 레퍼런스
> 생성자 레퍼런스는 메서드의 이름이 new 라는 점을 제외하면 메서드 레퍼런스와 유사하다.

예를 들어 Button::new는 Button 생성자를 가리키는 레퍼런스다.
```
Stream<Button> stream = labels.stream().map(Button::new);
```

## 변수 유효 범위
> 람다 표현식에서 해당 표현식을 감싸고 잇는 메서드나 클래스에 잇는 변수에 접근하고자 할때, 람다 표현식에서는 값이 변하지 않는 변수만 참조할 수 있다.

예를 들어 다음은 잘못된 사용법이다.
```
(count, text) -> while (count > 0) {
        count --; // 람다 표현식 내부에서 변수는 변경할 수 없다.
        System.out.println(text);
}
```

## 디폴트 메서드
> 자바8부터는 구체적인 구현을 담은 default method를 인터페에스에 허용하므로써, 기존 인터페이스(Collections과 같은 수년전 부터 사용되어오는 인터페이스들..)에 안전하게 새로운 메서드를 추가할 수 있다.

예를 들어 Collection의 슈퍼 인터페이스인 Iterable 인터페이스에 forEach 메서드를 추가하므로써 다음과 같이 람다식을 작성할 수 있다.
```
// java7
for (int i=0; i < list.size(); i++) {
        System.out.println(list.get(i));
}

// java8
list.forEach(System.out::println)
```

만약, default method가 추가되지 않았다면, Collection 인터페이스에 forEach와 같은 메서드가 신규로 추가될 경우
이전에 Collection을 구현하고 있던 모든 구현체들이 해당 메서드를 구현하기 전까지는 프로그램이 동작하지 않는 문제점이 있다.

간단히 예를 들자면 다음과 같은 인터페이스가 있을 때
```
interface Person {
        long getId();
        default String getName() { return "John Q. Public"; }
}
```
Person을 구현하는 클래스는 getId()는 항상 구현해야 하지만, getName()은 그대로 두거나 오버라이드 하는 방법 중 선택할 수 있다.

## 인터페이스 정적 메서드
> 자바8부터는 인터페이스에 정적 메서드를 추가할 수 있게 되었다.

기존에는 일반적으로 인터페이스를 구현하는 클래스에 정적 메서드를 선언했지만, 이제는 인터페이스에 선언해 다음과 같이 사용할 수도 있다.
```
public interface Path {
        public static Path get(String first, String... more) {
          return "..";
        }
}
```
