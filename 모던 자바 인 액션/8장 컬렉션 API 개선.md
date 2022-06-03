# 8장 컬렉션 API 개선

## 8.1 컬렉션 팩토리
자바9에선 컬렉션 객체를 쉽게 만들 수 있는 방법을 제공한다.

자바9 이전에는 아래와 같은 방식을 사용했었다.  
두 문자열을 저장하는 데도 많은 코드가 필요하다.
```java
ArrayList<String> friends = new ArrayList<>();
friends.add("frank");
friends.add("sole");
```

다음처럼 Arrays.asList()라는 팩토리 메서드를 통해 코드를 줄일수 있다.
```java
List<String> friends = Arrays.asList("frank","sole");
```
고정 크기의 요소를 만들었으므로 요소 갱신을 할 수 있으나 새 요소 추가나 삭제는 불가능하다.
이를 행하려 할땐 UnsuportedOperationException이 발생하게 된다.

이를 해결하기 위해선 어떻게 해야 할까?
자바9부턴 List.of라는 팩토리 메소드 객체로 쉽게 리스트를 만들 수 있다.  
이 컬렉션 객체는 변경,추가,삭제가 불가능하게 만들어져 있다. 따라서 컬렉션이 의도하지 않게
변하는 상황을 막을 수 있다. null 요소 또한 금지하므로 의도하지 않은 버그를 방지할 수 있다.
데이터 형식을 설정하거나 변환할 일이 없다면 편한 팩토리 메서드를 사용하는 것을 권장한다.

### 예시 코드
```java
List<String> friends = List.of("dhkstn", "frank", "josh");
Set<String> sets = Set.of("dhkstn", "frank", "josh");
Map<String, Integer> maps = Map.of("dhkstn", 10, "frank", 20, "josh", 30);
```

## 8.2 리스트와 집합처리
* removeIf - Predicate를 만족하는 요소를 제거한다.
* replcateAll - UnaryOperator를 이용해 요소를 바꾼다.
* sort - 리스트를 정렬

### removeIf
다음은 주어진 조건에 맞는 객체를 삭제하는 코드이다.
```java
for (Transaction transaction : transactions) {
            if(Character.isDigit(transaction.getTrader().getCity().charAt(0))) {
                transactions.remove(transaction);
            }
}
```
위 코드는 ConcurrentModificationException을 일으킨다. 왜냐면 내부적으로 for-each 루프는
Iterator 객체를 사용한다. 이렇게 되면 반복자의 상태와 실제 컬렉션의 상태가 동기화되지 않는
문제가 발생한다. 직접 iterator를 호출하고 remove를 함으로써 해결할 수 있다.  
이러한 패턴을 자바8에서는 removeIf로 스마트하게 바꿀 수 있다.

```java
transactions.removeIf(transaction -> Character.isDigit(transaction.getTrader().getCity().charAt(0)));
```
위 코드처럼 코딩하게 되면 코드가 단순해지고 에러도 예방할 수 있다. removeIf는 predicate를 인수로
받는다. 때로는 제거가 아닌 요소를 바꿔야할때가 있다. 이럴땐 replaceAll을 사용하면 된다.

### replaceAll
replcateAll은 UnaryOperator를 인수로 받아 사용할 수 있다.
```java
List<String> referenceCodes = Arrays.asList("a12", "C14", "b13");
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1));
```

## 8.3 맵 처리

### 8.3.1 forEach 메서드
자바8부턴 Map 인터페이스가 BiConsumer를 인수로 받는 forEach를 지원하므로 간단하게 구현할 수 있다.
```java
ageOfFriends.forEach((key, value) -> System.out.println(key + ":" + value));
```

### 8.3.2 정렬 메서드
두 개의 유틸리티를 이용하면 맵의 항목을 키, 밸류 기준으로 정렬할 수 있다.
* Entry.comparingByKey
* Entry.comparingByValue
```java
ageOfFriends.entrySet().stream()
                .sorted(Map.Entry.comparingByKey())
                .forEach(System.out::println);

ageOfFriends.entrySet().stream()
        .sorted(Map.Entry.comparingByValue())
        .forEachOrdered(System.out::println);
```

### 8.3.3 getOrDefault 메서드
찾으려는 키가 존재하지 않으면 NullPointerException이 발생하게 된다. 이를 방지하기 위해선
getOrDefault를 사용하면 된다. 키가 존재하지 않을땐 기본 값을 반환하게 된다.
```java
//getOrDefault(키,기본값)
System.out.println(ageOfFriends.getOrDefault("Olivia", 0));

//객체를 추가할때도 상태 변화를 위해 활용할 수 있다.
ageOfFriends.put("Hello",ageOfFriends.getOrDefault("Hello",0)+1);
```

> 추가 정보: HashMap의 성능
> 자바 8에선 HashMap의 구조를 바꿔 성능을 개선했다. 기존 맵의 항목은 키로 생성한 해시코드로 접근할 수 있는
> 버켓에 저장했다. 많은 키가 값은 해시코드를 반환하게 되면 O(n)의 시간이 걸려 성능이 저하되었다.
> 최근엔 버킷이 너무 커지면 O(log(n))의 시간이 소요되는 정렬된 트리를 이용해 동적 치환해 충돌이 일어나는
> 요소 반환 성능을 개선했다. 하지만 Key가 정렬된 트리를 지원하는 String, Number 여야만 한다.

### 8.3.4 계산 패턴
* computeIfAbsent - 제공된 키에 해당하는 값이 없다면, 키를 이용해 새 값을 계산하고 추가한다.
* computeIfPresent -  키가 존재하면 새 값을 계산하고 맵에 추가한다.
* compute - 제공된 키로 새 값을 계산하고 저장한다.
```java
//computeIfAbsent 제공된 키에 해당 값이 없으면 새 값을 계산하고 맵에 추가
Map<String, List<String>> friendsToMap = new HashMap<>();
friendsToMap.computeIfAbsent("null", name -> new ArrayList<>())
                .add("plus movie");
System.out.println(friendsToMap);
```

### 8.3.5 삭제 패턴
remove 메서드를 통해 지울 수 있다. 앞서 살펴본 내용이기에 지나가도록 한다.

### 8.3.6 교체 패턴
* replaceAll: BiFuction을 적용한 결과로 항목의 값을 교체한다.
* replace

```java
Map<String, String> favouriteMovies = new HashMap<>();
favouriteMovies.put("Raphael", "Star Wars");
favouriteMovies.put("Olivia", "james bond");
//replaceAll
favouriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
```

### 8.3.7 합침
두 개의 맵을 합칠수도 있다. putAll을 사용해 합칠 경우 중복 키들은 처리할 수 없다.
merge는 BiFunction을 이용해 value를 만들어낸다. 그러나 이 것도 3개 이상의 중복 밸류는
처리할 수 없다..
```java
Map<String, String> everyone2 = new HashMap<>(family);

//중복 밸류를 로직을 통해 처리할 수 있다.
friends2.forEach((k, v) -> everyone2.merge(k, v, (m1, m2) -> m1 + "&" + m2));
```

## 8.4 개선된 conCurrentHashMap
### 리듀스와 검색
conCurrentHashMap은 세 가지 연산을 지원한다.
* forEach
* reduce
* search

```java
//동시성 친화적이고, 동기화된 HashTable보다 읽기,쓰기 연산 기능이 월등이 높다.
ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
map.put("1", 1L);
map.put("#$", 3L);
long parallelism = 1;
Optional<Long> maxValue = Optional.ofNullable(map.reduceValues(parallelism, Long::max));
System.out.println(maxValue.get()); // 3
```