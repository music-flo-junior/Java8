## 타임 라인
- 일별 86,400초
- 1970년 1월 1일 자정이 기준
- Instant : 타임 라인의 한 시점
- Instant 와 Duration 클래스는 수정 불가이므로, multipliedBy나 minus 같은 모든 메서드는 새로운 인스턴스를 리턴한다.

## 지역 날짜
- 자바 API 시간 2종류 : '지역 날짜/시간(local date/time)' & '구역 시간(zoned time)'
- 일반적으로 '구역 시간'은 사용하지 않도록 권장한다. / 
- 구역 시간 예시 : 1976년 7월 16일 09:32:00 EDT (아폴로 11호 발사 일시?) 
- LocalDateTime에 타임존(tiem-zone)을 추가하면 ZonedDateTime이 된다.
  ZoneId는 일광 절약시간(DST, Daylight Saving Time)을 자동으로 처리해준다. // 서머타임을 의미 하는 듯
  LocalDateTime에 atZone()으로 시간대 정보를 추가하면, ZonedDateTime을 얻을 수 있다.
```
ZoneId zid = ZoneId.of("Asia/Seoul");
ZonedDateTime zdt = dateTime.atZone(zid);
```
- getDayOfWeek 은 DayOfWeek 열거 타입을 리턴한다.

## 날짜 조정기
- TemporalAdjuster 클래스는 일반적인 날짜 조정관련 여러 정적 메서드를 제공한다.
1) 지정 요일에 해당하는 다음 또는 이전 날짜
2) 주어진 날짜부터 시작해서 지정 요일에 해당하는 다음 또는 이전 날짜
3) 해당 월의 n 번째 지정 요일
4) 해당 월의 마지막 지정 요일
5) 메서드 이름에 기술된 날짜 

## 구역 시간
- 인터넷 할당 번화 관리 기관 (IANA)는 전 세계의 모든 시간데 데이터베이스를 유지하며, 자바는 이를 이용해 시간을 관리한다.
- 각 시간대에는 America/New_York 또는 Europe/Berlin 같은 ID 가 있다.
- 구역 시간을 사용해 '일광 절약 시간 (서머타임존?)'을 가로질러 날짜를 조정할 때에는 Duration이 아닌 Period를 사용해야 한다.

## 포맷팅과 파싱
- DateTimeFormatter 클래스는 날짜 / 시간 값을 출력하는 세 종류의 포맷터를 제공하낟.
1) 표준 포맷터
2) 로케일 종속 
3) 커스텀 패턴