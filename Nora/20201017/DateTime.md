* 새로운 날짜 및 시간 API
- 자바 1.0은 믿을 수 없을 만틈 고지식한 Date 클래스를 포함했다.
- 자바 1.1에서는 Calendar 클래스를 도입하고, Date 클래스의 메서드는 대부분 사용을 권장하지 않았다.
하지만 Calendar는 API가 뛰어나지 못했고, 인스턴스도 수정 가능했으며 윤초같은 이슈를 다루지 않았다.
- 자바 8에서 도입한 java.time API는 이전 시간들의 결함을 해결했고 앞으로 꽤 오랫동안 제 역할을 할 것이다.

* 핵심내용
- 모든 java.time 객체는 수정불가다.
- Instant는 타임 라인의 한 시점이다.(Date와 유사하다)
- 자바의 시간에서 하루는 정확히 86,400초다.
- Duration은 두 인스턴트 사이의 차이다.
- LocalDateTime은 시간대 정보를 포함하지 않는다.
- TemporalAdjuster의 메서드들은 일반적인 캘린더 계산을 처리한다. (특정 월의 첫 번째 화요일 찾기)
- ZoneDateTime은 주어진 시간대에서 특정 시점이다. (GregorianCalendar와 유사)
- 구역 시간을 앞으로 가게 할 때는 일광 절약 시간 변경을 고려하기 위해 Duration이 아닌 Period를 사용한다.
- DateTimeFormatter를 사용해 날짜와 시간을 해석한다.

* 01 타임라인
- 원자시계의 네트워크가 공식시간을 유지한다.
- 공식시간 유지장치들은 절대 시간을 지구의 회전과 줄곧 동기화한다. 
처음에는 공식 초를 미세하게 조정했지만, 1972년부터는 떄떄로 윤초를 삽입했다.
- 많은 컴퓨터 시스템에서는 윤초 직전에 인위적으로 시간을 느려지게 하거나 빨라지게 하여 일별 86,400초를 유지하는
스무딩 기술을 사용한다.
- 스무딩은 컴퓨터의 지역시간이 그리 정확하지 않고, 컴퓨터가 외부 시간 서비스와 자신의 시간을 동기화하기 때문에 가능하다.
- 자바의 날짜 및 시간 API 명세는 자바에서 다음과 같은 시간 척도를 사용하도록 요구한다
1. 일별 86,400초다
2. 각 일자의 정오에 공식 시간과 정확히 맞춘다.
3. 그 외에는 정확하게 정의된 방식으로 공식시간과 가깝게 맞춘다.
- 자바에서 Instant는 타임 라인의 한 시점을 나타낸다. Instant 값은 뒤로 10억년(Instant.MIN)까지 간다.
- 정적 메서드 호출 Instant.now()는 현재 인스턴트를 준다.
- 두 인스턴트를 비교하는 방법은 equals와 compareTo 메서드를 사용할 수 있다. 따라서 인스턴트를 타임스탬프로 사용할 수 있다.
- 두 인스턴트 사이의 차이를 알아낼려면 정적 메서드 Duration.between을 사용한다.
Instant start = Instant.now();
runAlgorithm();
Instant end = Instant.now();
Duration timeElapsed = Duration.between(start, end);
long millis = timeElapsed.toMillis();
- Duration은 내부 저장소로 long 값 이상을 요구한다. 초 수는 long에 저장되고 나노 초 수는 추가적인 int에 저장된다.
[Instant와 Duration용 산술연산]
- plus,minus : Instatnt 또는 Duration에 기간을 더하거나 뺀다.
- plusNanos, plusMillis, plusSeconds .. : Instant 또는 Duration에 주어진 시간 단위의 수를 더한다.
- minusNanos, minusMillis, minusSeconds .. : Instant 또는 Duration에 주어진 시간 단위의 수를 뺀다.
- multipliedBy, dividedBy, negated : 해당 Duration을 주어진 long값 또는 -1로 곱하거나 나누어서 얻은 Duration을 리턴한다.
Instant가 아닌 Duration만 크기 변경할 수 있다는 점에 유의한다.
- isZero, isNegative : Duration이 0 또는 음수인지 검사한다.
- Instant와 Duration 클래스는 수정 불가이므로, multipliedBy나 minus같은 모든 메서드는 새로운 인스턴스를 리턴한다.

* 02 지역날짜
- 새로운 자바 API에는 지역 날짜/시간과 구역 시간이라는 두 종류의 인간이 사용하는 시간이 있다.
- LocalDate는 연,월,일을 포함하는 날짜이다. LocalDate를 생성하는 데는 정적 메서드 now 또는 of를 사용할 수 있다.
[LocalDate 메서드]
- now,of : 현재 시각 또는 주어진 연,월,일로 부터 LocalDate를 생성하는 정적메서드다.
- plusDays, plusWeeks .. : 해당 LocalDate에 일, 주, 월, 연을 더한다.
- minusDays, minusWeeks .. : 해당 LocalDate에 일, 주, 월, 연을 뺀다. 
- plus, minus : Duration 또는 Period를 더하거나 뺀다.
- withDayOfMonth, withDayOfYear .. : 월 단위 일, 연 단위 일, 월 , 연을 주어진 값으로 변경한 새로운 LocalDate를 리턴한다.
- getDayOfMonth : 월 단위 일을 얻는다.
- getDayOfYear : 연 단위 일을 얻는다.
- getDayOfWeek : 요일을 얻는다.
- getMonth, getMonthValue : 월을 Month 열거 타입 값 또는 1~12 사이의 숫자로 얻는다.
- getYear : -999,999,999 ~ 999,999,999 사이의 연도를 얻는다.
- until : 두 날짜 사이에서 Period 또는 주어진 ChronoUnit 수를 구한다.
- isBefore, isAfter : 해당 LocalDate를 다른 LocalDate와 비교한다.
- isLeafYear : 해당 연도가 윤년이면 true를 리턴한다. (4로 나눌 수 있지만 100으로 나눌 수 없거나, 400으로 나눌 수 있음)

* 03 날짜 조정기
- TemporalAdjusters 클래스는 일반적인 조정에 사용하는 다수의 정적 메서드를 제공한다. 이러한 조정 메서드의 결과를 with 메서드에 전달할 수 있다.
[TemporalAdjusters 클래스의 날짜 조정기]
- next, previous : 지정 요일에 해당하는 다음 또는 이전 날짜
- nextOrSame, previousOfSame : 주어진 날짜부터 시작해서 지정 요일에 해당하는 다음 또는 이전 날짜
- dayOfWeekInMonth : 해당 월의 n번째 지정 요일
- lastInMonth : 해당 월의 마지막 지정 요일
- firstDayOfMonth(), firstDayOfNextMonth() .. : 메서드 이름에 기술된 날짜
- TemporalAdjuster 인터페이스를 구현하면 자신만의 조정기를 만들 수도 있다.
- 람다 표현식의 파라미터는 Temporal 타입이기 때문에 LocalDate로 캐스트 해야 한다.

* 04 지역 시간
- LocalTime의 인스턴스는 now 또는 of 메서드를 사용해 생성할 수 있다.
[LocalTime 메서드]
- now, of : 이들 정적 메서드는 현재 시각 또는 주어진 시, 분 그리고 선택적으로 초와 나노초로 부터 LocalTime을 생성한다.
- plusHours, plusMinutes .. : 해당 LocalTime에서 시, 분 , 초 또는 나노초를 더한다.
- minusHours, minusMinutes .. : 해당 LocalTime에서 시, 분 , 초 또는 나노초를 뺀다.
- plus, minus : Duration을 더하거나 뺀다.
- withHour, withMinute : 시, 분, 초 또는 나노초가 주어진 값으로 변경된 새로운 LocalTime을 리턴한다.
- getHour, getMinute : 해당 LocalTime의 시, 분, 초 또는 나노초를 얻는다.
- toSecondOfDay, toNanoOfDay : 자정과 해당 LocalTime 사이의 초 또는 나노초 수를 리턴한다.
- isBefore, isAfter : 해당 LocalTime을 다른 LocalTime과 비교한다.
- 날짜와 시간을 표현하는 LocalDateTime 클래스가 있다. 이 클래스는 고정 시간대에서 시간의 한 점을 저장하는데 적합하다.
- 일광 절약 시간을 포함하는 계산을 해야 하거나 서로 다른 시간대에 있는 사용자들을 다뤄야 한다면 ZonedDateTime 클래스를 사용한다.

* 05 구역 시간
- 시간대는 지구의 불규칙적인 회전에 기인한 복잡한 문제들보다도 다루기 까다롭다. 다른 세계에서는 경계가 불규칙하고 이동하는 시간대와 일광 절약 시간이 있다.
- 인터넷 할당 번호 관리 기관은 전 서계의 알려진 모든 시간대 데이터베이스를 유지하며, 이 데이터베이스는 1년에 몇차례 업데이트 된다.
대부분의 업데이트는 일광 절약 시간 변경 규칙을 다룬다. 자바는 IANA 데이터를 사용한다.
- 각 시간대에는 America/New_York 또는 Europe/Berlin과 같은 ID가 있다. 이용 가능한 모든 시간대를 얻으려면 ZoneId.getAvailableIds를 호출한다.
- 정적 메서드 ZoneId.of(id)는 시간대 ID를 넘겨주면 zoneId 객체를 돌려준다.
- 이 객체를 사용해 local.atZone(zoneId) 형식으로 호출하면 LocalDateTime 객체를 ZoneDateTime 객체로 변환할 수 있다.
- ZoneDateTime의 메서드 중 상당 수는 LocalDateTime의 메서드들과 같다.
[ZonedDateTime 메서드]
- now, of, ofInstant : 이들 정적 메서드는 현재 시각 또는 주어진 시, 분 그리고 선택적으로 초와 나노초로 부터 LocalTime을 생성한다.
- plusHours, plusMinutes .. : 해당 LocalTime에서 시, 분 , 초 또는 나노초를 더한다.
- minusHours, minusMinutes .. : 해당 LocalTime에서 시, 분 , 초 또는 나노초를 뺀다.
- plus, minus : Duration을 더하거나 뺀다.
- withDayOfMonth, withDayOfYear .. : 특정 시간 단위가 주어진 값으로 변경된 새로운 ZoneDateTime을 리턴한다.
- withZoneSameInstant, withZoneSameLocal : 주어진 시간대에서 동일한 인스턴스 또는 지역시간을 나타내는 새로운 ZoneDateTime을 리턴한다.
- getDayOfMonth : 월 단위 일을 얻는다.
- getDayOfYear : 연 단위 일을 얻는다.
- getDayOFWeek : 요일을 얻는다.
- getMonth, getMonthValue : 월은 Month 열거 타입 값 또는 1 ~ 12 사이의 숫자 값으로 얻는다.
- getYear : -999,999,999 ~ 999,999,999 사이의 연도를 얻는다.
- getHour, getMinute, getSecond, getNano : 해당 ZoneDateTime의 시, 분, 초 또는 나노초를 얻는다.
- getOffset : ZoneOffset 인스턴스로 UTC로부터의 오프셋을 얻는다. 오프셋은 -12:00 ~ +14:00까지 달라질 수 있다.
몇몇 시간대는 소수점 오프셋을 포함한다. 오프셋은 일광 절약 시간과 함께 변한다.
- toLocalDate, toLocalTime, toInstant : 각각 지역 날짜, 지역 시간, 대응 인스턴트를 돌려준다.
- isBefore, isAfter : 해당 ZonedDateTime을 다른 ZonedDateTime과 비교한다.

* 06 포맷팅과 파싱
- DateTimeFormatter 클래스는 날짜/시간 값을 출력하는 세 종류의 포맷터를 제공한다.
1. 미리 정의된 표준 포맷터
2. 로케일 종속 포맷터
3. 커스텀 패턴을 이용하는 포맷터
[미리 정의된 포맷터]
- BASIC_ISO_DATE : 구분자 없는 연,월,일,시간대 오프셋 - 19690716-0500
- ISO_LOCAL_DATE / ISO_LOCAL_TIME / ISO_LOCAL_DATE_TIME : -,:,T 구분자 사용 - 1689-07-16, 06:32:00
- ISO_OFFSET_DATE / ISO_OFFSET_TIME / ISO_OFFSET_DATE_TIME : ISO_LOCAL_XXX와 유사하지만 시간대 오프셋 포함
- ISO_ZONED_DATE_TIME : 시간대 오프셋과 시간대 ID 포함
- ISO_INSTANT : 시간대 ID Z로 표기하는 UTC 형식
- ISO_DATE / ISO_TIME / ISO_DATE_TIME : 시간대 정보가 선택적임
- ISO_ORDINAL_DATE : localDate에 해당하는 연도와 연 단위 일
- ISO_WEEK_DATE : localDate에 해당하는 연, 주, 요일
- RFC_1123_DATE_TIME : 이메일 타임스탬프 표준으로, RFC 822에서 표준화 되었고 RFC 1123에서 연도에 4자리 숫자를 사용하도록 업데이트됨
- 표준 포맷터 중 하나를 사용하려면, 단순히 format 메서드를 호출하면 된다.
string formatted = DateTimeFormatter.ISO_DATE_TIME.format(apollo11launch);
- 표준 포맷처는 대부분 기계가 읽을 수 있는 타임스탬프용으로 만들어졌다. 사람이 읽을 수 있는 날짜와 시간을 표현하려면 로케일 종속 포맷터를 사용한다.
[로케일 종속 포맷팅 형식]
- SHORT : 7/16/69 - 9:32 AM
- MEDIUM : Jul 16, 1969 9:32:00 AM
- LONG : July 16, 1969 9:32:00 AM EDT
- FULL : Wednesday, July 16, 1969 9:32:00 AM EDT
- 마지막으로 패턴을 지정해서 자신만의 날짜/시간 형식을 만들 수 있다.
formatter = DateTimeFormatter.ofPattern("E yyyy-MM-dd HH:mm");

* 07 레거시 코드와 상호동작
- 새로 생겨난 자바의 날짜 및 시간 API는 기존 클래스들, 특히 널리 사용되고 있는 GregorianCalendar 그리고 TimeStamp와 상호동작 해야한다.
[java.time 클래스들과 레거시 클래스 사이의 변환]
1.Instant <> java.util.Date : Date.from(instant) / date.toInstant()
2.ZoneDateTime <> java.util.GregorianCalendar : GregorianCalendar.from(zoneDateTime) / cal.toZonedDateTime()
3.Instant <> java.sql.Timestamp : Timestamp.from(instant) / Timestamp.ToInstant()
...