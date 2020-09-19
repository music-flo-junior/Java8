# Interface + Lambda

## 메소드 참조
```java
Arrays.sort(strings, (x, y) -> x.compareToIgnoreCase(y));
Arrays.sort(strings, String::compareToIgnoreCase);

list.removeIf(e -> e == null);
list.removeIf(Objects::isNull);
```

## 생성자 참조
```java
List<String> names = ...;
Stream<Employee> stream = names.stream().map(Employee::new);

map 메소드는 메소드를 적용하고 모든 결과를 모은다.
names.stream()에 String 객체가 담겨 있으므로 컴파일러는 Employee::new가 Employee(String) 생성자를 가리킨다는 사실을 알게 된다.

배열 생성자 참조는 자바의 한계를 극복하는 데 유용하다. 자바에서는 제네릭 타입으로 배열을 생성할 수 없다.
그래서 Stream.toArray와 같은 메소드는 Object 배열을 반환한다.
Object[] employees = stream.toArray();

하지만 개발자는 객체의 배열이 아니라 직원의 배열을 원한다.
이 문제를 해결하기 위해 toArray의 또 다른 버전은 다음과 같이 생성자 참조를 받는다.
Employee[] buttons = stream.toArray(Employee[]::new);
```

## 변수 유효 범위
람다 표현식에는 값이 변하지 않는 변수만 참조할 수 있다.<br>
이런 이유 때문에 람다 표현식은 변수가 아니라 값을 캡처한다고 말하기도 한다.<br>
그리고 캡쳐한 변수를 변경할 수 없다. 
```java
(A)
for (int i = 0; i < n; ++i) {
    new Thread(
        () -> System.out.println(i)
        // 오류 - i를 캡처할 수 없다.
    ).start();
}

(B)
for (String value : values) {
    new Thread(
        () -> System.out.println(value)
        // value를 캡쳐할 수 있다.
    ).start();   
}

(B)에서는 각 반복마다 새로운 변수 value가 생성되고, values에 있는 값을 할당받는다.
(A)에서는 i의 유효 범위가 전체 루프다.
```


