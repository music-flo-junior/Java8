# 날짜와 시간 API
* java.time 객체는 모두 immutable
* TemporalAdjuster 메소드는 일반적인 달력 계산 처리
    > 특정 월의 첫 번째 화요일과 같은 것들
* 날짜와 시간 서식을 지정하고 해석하는 데는 DateTimeFormatter를 사용

## 기계 시간
* Instant.now() : 현재 UTC(GMT) 리턴

## 사람 시간
* LocalDateTime.now() : 현재 시스템 Zone에 해당하는(로컬) 일시 리턴
* LocalDateTime.of(int, Month, int, int, int, int) : 로컬의 특정 일시 리턴
* ZonedDateTime.of(int, Month, int, int, int, int, ZoneId): 특정 Zone의 특정 일시 리턴

## Caution
1월 31일에 한달을 더하거나 3월 31일에서 한달을 빼면 2월 31일이 아니고 2월 29일을 리턴해준다.
```java
LocalDate.of(2016, 1, 31).plusMonths(1)
LocalDate.of(2016, 3, 31).minusMonths(1)
```