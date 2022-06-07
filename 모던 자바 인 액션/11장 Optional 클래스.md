# 11장 Null 대신 Optional 클래스
## 11.1 값이 없는 상황을 어떻게 처리할까?
### 11.1.1 보수적인 자세로 NullPointerException 줄이기
필요한 곳에서 다양한 null을 통해 NPE를 해결한다. 모든 변수가 null인지 의심하므로 변수를
접근할 때마다 중첩된 if가 추가되면서 코드 들여쓰기 수준이 증가한다. 이러한 반복 패턴을
깊은 의심이라고 부른다. 이 때문에 유지 보수가 어려워질 수 있다.

### 11.1.2 null 때문에 발생하는 문제
* 에러의 근원 : NPE는 자바에서 가장 흔하게 발생하는 에러이다. 
* 코드를 어지럽힌다 : 중첩된 null 확인 코드를 추가해야 하므로 null 때문에 코드 가독성이 떨어진다. 
* 아무 의미가 없다 : null은 아무 의미도 표현하지 않는다. 값이 없음을 표현하는 방식으로는 적절하지 않다. 
* 자바 철학에 위배된다 : 자바는 개발자로부터 모든 포인터를 숨겼다. 하지만 null 포인터는 예외이다. 
* 형식 시스템에 구멍을 만든다 : 모든 참조 형식에 null을 할당할 수 있다. 이런 식으로 null이 할당되기 시작하면서 시스템의
다른 부분으로 null이 퍼졌을 때 애초에 null이 어떤 의미로 사용되었는지 알 수 없다.

### 11.1.3 다른 언어의 null 대신 무얼 사용하나?
최신 그루비 언어과 코틀린은 안전 네비게이션 연산자 ?.를 도입해서 null 문제를 해결했다.
```groovy
def carInsuranceName = person?.car?.insurance?.name
```

## 11.2 Optional 클래스 소개
자바 8은 하스켈과 스칼라의 영향을 받아서 java.util.Optional라는 새로운 클래스를 제공한다. Optional은 선택형값을 캡슐화하는 클래스다. 
값이 있으면 Optional 클래스는 값을 감싼다.  
반면 값이 없으면 Optional.empty 메서드로 Optional을 반환한다. Optional.empty는 Optional의 특별한 싱글턴 인스턴스를 반환하는 정적 팩토리 메서드이다.

null참조와 Optional.empty에는 차이가 있다. 
null을 참조하려하면 NPE이 발생하지만 Optional.empty()는 Optional 객체이므로 이를 다양한 방식으로 활용할 수 있다.  

Optional을 사용하면 모델의 의미가 더 명확해진다. Optional을 이용하면 값이 없는 사오항이 우리 데이터에 문제가 있는지 아닌지에 대한
것을 명확히 구분할 수 있다. Optional의 역할은 더 쉬운 API를 설계하도록 돕는 것이다.

## 11.3 Optional 적용 패턴
### 11.3.1 Optional 객체 만들기
#### 빈 Optional
```java
Optional<Car> optCar = Optional.empty()
```
#### null이 아닌 값으로 Optional 만들기
```java
Optional<Car> optCar = Optional.of(car);
```

#### null 값으로 Optional 만들기'
car가 null이어도 동작한다.
```java
Optional<Car> optCar = Optional.ofNullable(car);
```

### 11.3.2 맵으로 Optional의 값을 추출하고 반환하기
스트림의 map과 비슷하게 각 요소에 제공된 함수를 적용한다. 
Optional이 비어있으면 아무 일도 일어나지 않는다. map을 통해 반환되는 객체 또한 Optional에 감싸져 반환된다.
```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName());
```

### 11.3.3 flatMap으로 Optional 객체 연결 
map 메서드로 반환되는 값은 Optional로 감싸짐을 확인할 수 있다.
그렇다면 반환하는 객체가 이미 Optional 객체일 경우 반환되는 타입이 Optional<Optional>이다. 
두번 감싸진 Optional로 받고 싶지 않다면 flatMap 메서드를 사용하면 된다. 
flatMap 메서드는 전달된 Optional 객체의 요소에 대해 새로운 Optional 반환한다.
```java
public String getCarInsuranceName(Optional<Person> person) {
        return person.flatMap(Person::getCar)
        .flatMap(Car::getInsurance)
        .map(Insurance::getName);
        }
```

> 도메인 모델에 Optional 사용했을 때 데이터를 직렬화할 수 없는 이유  
> Optional 클래스는 필드 형식으로 사용할 것을 가정하지 않았으므로 Serializable 인터페이스를 구현하지 않는다.  
> 따라서 직렬화 모델을 사용하게 되면 문제가 생길 수 있다.  
> 만약 직렬화 모델이 필요하다면 변수는 일반 객체로 두되 Optional 값을 반환받는 방식을 권장한다.

### 11.3.4 Optional 스트림 조작
자바 9에선 Optional을 포함하는 스트림을 쉽게 처리할 수 있도록 Optional에 stream 메서드를 추가했다.
Optional 스트림을 값을 가진 스트림으로 변화할때 유용하다.
```java
public Set<String> getCarInsuranceNames(List<Person> persons) {
    return persons.stream()
        .map(Person::getCar) //optinal로 변환
        .map(optCar -> optCar.flatMap(Car::getInsurance))
        .map(optInsurance -> optInsurance.map(Insurance::getName)) //Optional<String> 반환
        .flatMap(Optional::stream) //Stream<Optional<String>>을 Stream<String>로 변환
        .collect(toSet());
}
```
### 11.3.5 디폴트 액션과 Optional 언랩
* get : 값을 읽는 가장 간단한 메서드면서 동시에 가장 안전하지 않은 메서드이다. 값이 없으면 NoSuchElementException을 뱉기에 값이 반드시 있다고 가정할 수 있는 상황이 아니면 get메서드를 사용하지 말자 
* orElse : Optional이 값을 포함하지 않을 때 기본값을 제공할 수 있다. 
* orElseGet(Supplier<? extends T> other) : orElse 메서드에 대응하는 게으른 버전의 메서드이다. Optional에 값이 없을 때만 Supplier가 실행된다. 
* orElseThrow(Supplier<? extends X> exceptionSupplier) : Optional이 비어있을 때 예외를 발생시킬 수 있으며, 발생시킬 예외의 종류를 선택할 수 있다. 
* ifPresent(Consumer<? super T> consumer) : 값이 존재할 대 인수로 넘겨준 동작을 실행할 수 있다. 값이 없으면 아무일도 일어나지 않는다. 
* ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction) : Optional이 비었을 때 실행할 Runnable을 인수로 받는다

## Optional 제공 함수

|      메서드      |                           파라미터와 리턴값                            |
|:-------------:|:--------------------------------------------------------------:|
|     empty     |                       빈 Optional 인스턴스 반환                       |
|    filter     | 값이 존재하며 프레디케이트와 일치하면 값을 포함하는 Optional을 반환하고 아니면 빈 Optional을 반환 |
|    flatMap    |  값이 존재하면 인수로 제공된 함수를 적용 결과 Optional을 반환하고 아니면 빈 Optional을 반환   |
|      get      |    값이 존재하면 Optional을 반환하고 아니면 NoSuchElementException을 발생시킴     |
|   ifPresent   |         값이 존재하면 지정된 Consumer를 실행하고, 없으면 아무 일도 일어나지 않음          |
| ifPresentElse |         값이 존재하면 지정된 Consumer를 실행하고, 없으면 아무 일도 일어나지 않음          |
|   isPresent   |                     값이 존재하면 true 아니면 false                     |
|      map      |                         제공된 매핑 함수를 적용함                         |
|      of       |       값이 존재하면 Optional을 반환하고 아니면 NullPointerException 발생       |
|  ofNullable   |           값이 존재하면 Optional을 반환하고 아니면 빈 Optional을 반환            |
|      or       |    값이 존재하면 같은 Optional을 반환하고 아니면 Supplier에서 만든 Optional을 반환    |
|    orElse     |                  값이 존재하면 값을 반환하고 아니면 기본값을 반환함                  |
|   orElseGet   |          값이 존재하면 같은 값을 반환하고 아니면 Supplier에서 제공하는 값을 반환          |
|  orElseThrow  |       값이 존재하면 같은 값을 반환하고 아니면 Supplier에서 만든 Optional을 반환        |
|    stream     |              값이 존재하면 stream을 반환하고 아니면 빈 stream 반환              |

### 마치며
* 자바는 값의 존재 유무를 표현할 수 있는 클래스 java.util.Optional<T>를 제공한다.
* 팩토리 메서드를 통해 Optional을 생성할 수 있다.
* Optional은 스트림과 비슷한 연산을 수행하는 메서드를 지원한다.
* Optional을 통해 예상치 못한 null 예외를 방지할 수 있다.