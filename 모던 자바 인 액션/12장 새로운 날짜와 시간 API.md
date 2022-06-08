# 12장 새로운 날짜와 시간 API
### 자바 8 이전 API의 문제
* Date 클래스는 직관성, 유용성이 떨어지게 설계되었다.
* Date를 보완하기 위한 Calendar 또한 설계 문제로 에러를 일으킨다.
* DateFormat 또한 스레드에 안전하지 못했다.
* 가변 클래스였기에 유지보수가 아주 어려워졌다.

## 12.1 LocalDate, LocalTime, Instant, Duration, Period 클래스
### 12.1.1 LocalDate와 LocalTime 사용
