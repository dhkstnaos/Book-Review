# ch3. 람다 표현식

## 람다란 무엇인가?
람다란 메서드로 전달할 수 있는 익명 함수를 단순화한 것이다.  
람다 표현식은 이름은 없다. 그러나 파라미터, 반환 형식, 예외 리스트를 가질 수 있다.  

#### 람다의 특징
* 익명 - 메서드가 정해져 있지 않아 익명의 특징을 가진다.
* 함수 - 특정 클래스에 종속되지 않아 함수라고 부른다.
* 전달 - 메서드 인수로 전달하거나 변수로 저장이 가능하다.
* 간결성 - 익명 클래스는 부피가 크지만 람다는 그렇지 않다.

```java
inventory.sort(new Comparator<Apple>() {
            @Override
            public int compare(Apple o1, Apple o2) {
                return o1.getPrice() - o2.getPrice();
            }
});
===============람다로 변환하면 길이가 훨씬 짧다==============     
inventory.sort((Apple a1, Apple a2) -> a1.getPrice() - a2.getPrice());
```

#### 람다의 3가지 구성
* 파라미터 리스트 -> Ex) (Apple a1, Apple a2) 메서드 파라미터
* 화살표 -> 는 파라미터와 바디를 구분하는 구분자이다.
* 람다 바디 -> 람다의 반환값에 해당하는 표현식이다.


#### 람다의 5가지 표현법
```java
=> 파라미터 하나를 가지며 int를 반환한다. 람다에는 Return이 함축되어 있다.
(String s) -> s.length() 

=> 파라미터 하나를 가지면서 boolean을 return한다.
(Apple a) -> a.getWeight() > 150 

=> 파라미터 두 개를 가지나 return 값은 없다.
(int x,int y) -> {
    System.out.println("Result:");
    System.out.println(x+y);
}

=> 파라미터는 없으나 int를 반환한다.
() -> 42

=> 파라미터 두 개를 가지고 int를 반환한다.
(Apple a1,Apple a2) -> a1.getWeight()-a2.getWeight())
```

## 함수형 인터페이스 모음
|인터페이스|파라미터와 리턴값|설멍|
|:---:|:---:|:---:|
|Predicate<T>|T -> boolean|불리언 표현|
|Consumer<T>|T -> void|객체 소비|
|Function<T, R>|T -> R|객체 선택 및 추출|
|Supplier<T>|() -> T|객체 생성|
|UnaryOperator<T>|T -> T|객체 선택 및 추출|
|BinaryOperator<T>|(T, T) -> T|두 값을 조합|
|BiPredicate<L, R>|(T, U) -> boolean|불리언 표현|
|BiConsumer<T, U>|(T, U) -> void|객체 소비|
|BiFunction<T, U, R>|(T, U) -> R|두 객체 비교|
  
#### 함수형 인터페이스의 예외 동작
함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않는다.
따라서 예외를 만드려면 직접 함수형 인터페이스를 정의하거나 람다를 try-catch 블록으로 감싸야 한다.
```java
Function<BufferedREader, String> f = (BufferedREader b) -> {
  try {
      return b.readLine();
  }
  catch(IOException e) {
      throw new RuntimeException(e);
  }
};
```
  
#### 메서드 참조
메서드 참조를 이용해 기존 메서드 정의를 활용해 람다처럼 전달할 수 있다.
이를 활용하면 코드 가독성을 높일 수 있다.
  
```java
(Apple apple) -> apple.getWeight()

Apple::getWeight
```
#### 메서드 참조 방법 3가지
* 정적 메서드 참조 - Integer::parseInt로 표현할 수 있다.
```java
(args) -> ClassName.staticMethod(args)
  
ClassName::staticMethod
```
* 인스턴스 메서드 참조 - String::length 같은 걸로 표현할 수 있다.
```java
(arg, result) -> arg.instanceMethod(result)
  
ClassName::instanceMethod
```
* 기존 객체의 인스턴스 메서드 참조 - Transaction 안에 getValue 메서드가 있다면 가져올 수 있다.
```java
(args) -> expression.instanceMethod(args)
  
expression::instanceMethod
```

#### 생성자 참조
ClassName::new처럼 new 키워드를 이용해 생성자 참조를 만들 수 있다.
```java
Supplier<Apple> c1 = Apple::new;
Apple a1 = c1.get();
```
            
#### 람다, 메서드 참조 활용하기
```java
//익명 클래스
inventory.sort(new Comparator<Apple>() {
        @Override
        public int compare(Apple o1, Apple o2) {
                return o1.getPrice() - o2.getPrice();
        }
});

//람다 표현식
inventory.sort((Apple o1, Apple o2) -> o1.getPrice() - o2.getPrice());

//코드 간소화
inventory.sort(comparing(a -> a.getPrice()));

//메서드 참조로 가독성 향상
inventory.sort(comparing(Apple::getPrice));            
```
