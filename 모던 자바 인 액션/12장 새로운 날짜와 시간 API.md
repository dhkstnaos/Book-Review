# 12장 새로운 날짜와 시간 API
### 자바 8 이전 API의 문제
* Date 클래스는 직관성, 유용성이 떨어지게 설계되었다.
* Date를 보완하기 위한 Calendar 또한 설계 문제로 에러를 일으킨다.
* DateFormat 또한 스레드에 안전하지 못했다.
* 가변 클래스였기에 유지보수가 아주 어려워졌다.

## 12.1 LocalDate, LocalTime, Instant, Duration, Period 클래스
### 12.1.1 LocalDate와 LocalTime 사용
LocalDate는 시간을 제외한 날짜를 표현하는 불변 객체이다. 어떤 시간대 정보도 포함하지 않는다.  
정적 팩토리 메서드 of로 객체를 만들 수 있다.

팩토리 메서드 now를 통해 현재 날짜 정보를 얻을 수도 있다.
```java
LocalDate date = LocalDate.of(2014, 3, 10);
int year = date.getYear(); // 2014
Month month = date.getMonth(); // MARCH
int day = date.getDayOfMonth(); // 18
DayOfWeek dow = date.getDayOfWeek(); // TUESDAY
int len = date.lengthOfMonth(); // 31 (3월의 길이)
boolean leap = date.isLeapYear(); // false (윤년이 아님)
System.out.println(date);

LocalDate now = LocalDate.now(); // 현재 날짜 정
```

시간에 대한 정보는 LocalTime 클래스로 표현할 수 있다. LocalTime도 정적 메서드 of로 인스턴스를 만들 수 있다.  
```java
LocalTime time = LocalTime.of(13, 45, 20); // 13:45:20
int hour = time.getHour(); // 13
int minute = time.getMinute(); // 45
int second = time.getSecond(); // 20
System.out.println(time);
```

parse 라는 정적 메서드를 활용해 문자열을 통해 객체를 만들수도 있다.
```java
LocalDate date = LocalDate.parse("2022-06-08");
LocalTime time = LocalTime.parse("17:55:55");
```

### 12.1.2 날짜와 시간 조합
LocalDateTime은 LocalDate와 LocalTime을 쌍으로 갖는 복합 클래스다. 날짜와 시간을 모두 표현하고, of를 통해 만들 수 있다. 
atDate, atTime을 통해서도 만들 수 있다.
```java
LocalDateTime dt1 = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45, 20); // 2014-03-18T13:45
LocalDateTime dt2 = LocalDateTime.of(date, time);
LocalDateTime dt3 = date.atTime(13, 45, 20);
LocalDateTime dt4 = date.atTime(time);
LocalDateTime dt5 = time.atDate(date);
```

toLocalDate 또는 toLocalTime 메서드로 LocalDate, LocalTime 인스턴스를 추출할 수 있다.
```java
LocalDate date1 = dt1.toLocalDate();
LocalTime time1 = dt1.toLocalTime();
```

### 12.1.3 Instant 클래스 : 기계의 날짜와 시간
사람은 보통 주,날짜,시간,분으로 날짜와 시간을 얘기한다. java.time.Instant는 UTC 시간(1970년 1월 1일 0시 0분 0초)을 기준으로
특정 지점까지의 시간을 초로 표현한다. 팩토리 메서드 ofEpochSecond에 초를 넘겨 인스턴스를 생성할 수 있다.  
나노초를 제공해 두번째 인수를 사용해 나노초 시간을 정의할 수 있다.(0~999,999,999 사이의 값)
```java
Instant instant = Instant.ofEpochSecond(44 * 365 * 86400);
Instant now = Instant.now();
```

### 12.1.4 Duration과 Period 정의
지금까지 살펴본 모든 클래스는 Temporal 인터페이스를 구현하는데, 
Temporal 인터페이스는 특정 시간을 모델링하는 객체의 값을 어떻게 읽고 조작할지 정의한다.  
두 객체 사이의 지속시간을 활용할 수 있는 Duration 클래스가 있다.
Duration은 초와 나노초로 시간을 표현하기에 LocalDate를 전달할 수 없다.  
년,월,일 시간을 표현할땐 Perior 클래스의 between이라는 팩토리 메서드를 활용하면 차이를 알 수 있다.