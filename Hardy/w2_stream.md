# Stream API

## Stream
* sequence of elements supporting sequential and parallel aggregate operations
* 데이터를 담고 있는 저장소(컬렉션)이 아니다.
* 스트림이 처리하는 데이터 소스를 변경하지 않는다.
* 스트림으로 처리하는 데이터는 오직 한번만 처리한다.
* 무제한일 수도 있다. (Short Circuit 메소드를 사용해서 제한할 수 있다.)
* 중개 오퍼레이션은 근본적으로 Lazy하다.
* 손쉽게 병렬 처리할 수 있다.

## Stream Pipeline
* 0 또는 다수의 중개 오퍼레이션(intermediate operation)과 1개의 종료 오퍼레이션(terminal operation)으로 구성한다.
* 스트림의 데이터 소스는 오직 터미널 오퍼레이션을 실행할 때만 처리한다.

## 중개 오퍼레이션
* Stream을 리턴한다.
* Stateless / Stateful 오퍼레이션으로 더 상세히 구분할 수 있다.
    > 대부분 Stateless지만 distinct나 sorted처럼 이전 소스 데이터를 참조해야 하는 오퍼레이션은 Stateful이다.
* filter, map, limit, skip, sorted ...

## 종료 오퍼레이션
* Stream을 리턴하지 않는다.
* collect, allMatch, count, forEach, min, max ...

## groupingBy, mapping 예제
* 현재도 자주 사용하는 filter, map, forEach 등은 다루지 않는다.
* (내가 만든거라 부족한 부분이 있을 수 있음)
```java
public class City {
    private String state;
    private String name;
    private int population;

    public City(String state, String name, int population) {
        this.state = state;
        this.name = name;
        this.population = population;
    }

    public String getState() {
        return state;
    }

    public String getName() {
        return name;
    }

    public int getPopulation() {
        return population;
    }
}
```
```java
public class GroupingByEx {

    public static void main(String[] args) {
        City suwon = new City("경기도", "수원", 1000);
        City osan = new City("경기도", "오산", 500);
        City ilsan = new City("경기도", "일산", 1500);
        City busan = new City("경상남도", "부산", 5000);
        City pohang = new City("경상남도", "포항", 4000);

        List<City> cityList = new ArrayList<>();
        cityList.add(suwon);
        cityList.add(osan);
        cityList.add(ilsan);
        cityList.add(busan);
        cityList.add(pohang);

        Map<String, Long> stateCountMap = cityList.stream().collect(
                groupingBy(City::getState, counting())
        );
        System.out.println(stateCountMap.get("경기도"));

        Map<String, Integer> stateToCityPopulation = cityList.stream().collect(
                groupingBy(
                        City::getState,
                        summingInt(City::getPopulation)
                )
        );
        System.out.println(stateToCityPopulation.get("경기도"));

        Map<String, Optional<City>> stateToLargestCity = cityList.stream().collect(
                groupingBy(
                        City::getState,
                        maxBy(Comparator.comparing(City::getPopulation))
                )
        );

        stateToLargestCity.get("경기도")
                .ifPresentOrElse(
                        city -> System.out.println(city.getName()),
                        () -> System.out.println("없음")
                );

        stateToLargestCity.get("경상남도")
                .ifPresentOrElse(
                        city -> System.out.println(city.getName()),
                        () -> System.out.println("없음")
                );

        Map<String, Set<String>> stateToName = cityList.stream().collect(
                groupingBy(
                        City::getState,
                        mapping(City::getName, toSet())
                )
        );
        System.out.println(stateToName.get("경기도"));

        Map<String, IntSummaryStatistics> stateToCityPopulationSummary =
                cityList.stream().collect(
                        groupingBy(
                                City::getState,
                                summarizingInt(City::getPopulation))
                );
        System.out.println(stateToCityPopulationSummary.get("경기도"));
    }

}
```

## 병렬 스트림 예제
```java
List<OnlineClass> springClasses = new ArrayList<>();
springClasses.add(new OnlineClass(1, "spring boot", true));
springClasses.add(new OnlineClass(2, "spring data jpa", true));
springClasses.add(new OnlineClass(3, "spring mvc", false));
springClasses.add(new OnlineClass(4, "spring core", false));
springClasses.add(new OnlineClass(5, "rest api development", false));

List<OnlineClass> javaClasses = new ArrayList<>();
javaClasses.add(new OnlineClass(6, "The Java, Test", true));
javaClasses.add(new OnlineClass(7, "The Java Code manipulation", true));
javaClasses.add(new OnlineClass(8, "The Java, 8 to 11", false));

List<List<OnlineClass>> hardyEvents = new ArrayList<>();
hardyEvents.add(springClasses);
hardyEvents.add(javaClasses);

hardyEvents.stream()
        .flatMap(Collection::stream)
        .forEach(cls -> System.out.println(cls.getId() + " " + cls.getTitle()));

hardyEvents.parallelStream()
        .flatMap(Collection::stream)
        .forEach(cls -> System.out.println(cls.getId() + " " + cls.getTitle()));
```
```java
List<String> words = Arrays.asList("하디", "스프링", "봄", "애플", "아이패드", "아이폰", "셀토스", "쏘카", "그린카", "자바");
Map<Integer, Long> shortWordCount = words.parallelStream()
        .filter(s -> s.length() < 3)
        .collect(
                groupingBy(
                        String::length,
                        counting()
                )
        );
System.out.println(shortWordCount.getOrDefault(3, 0L));
System.out.println(shortWordCount.get(2));
System.out.println(shortWordCount.get(1));

ConcurrentMap<Integer, List<String>> result = words.parallelStream()
        .collect(Collectors.groupingByConcurrent(String::length));
System.out.println(result.getOrDefault(5, Collections.EMPTY_LIST));
System.out.println(result.get(4));
System.out.println(result.get(3));
System.out.println(result.get(2));
System.out.println(result.get(1));

ConcurrentMap<Integer, Long> wordCounts = words.parallelStream()
        .collect(Collectors.groupingByConcurrent(String::length, counting()));
System.out.println(wordCounts.getOrDefault(5, 0L));
System.out.println(wordCounts.get(4));
System.out.println(wordCounts.get(3));
System.out.println(wordCounts.get(2));
System.out.println(wordCounts.get(1));
```