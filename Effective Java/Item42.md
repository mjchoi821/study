## Item 42. 익명 클래스보다는 람다를 사용하라

<br>

### 람다식(lambda expression)

- 자바8 부터 지원
- 메서드를 하나의 식(expression) 으로 표현한 것
- 함수형 인터페이스의 인스턴스를 람다식으로 사용

<br>

### 코드의 발전 과정

##### 람다식 이전의 코드(익명 클래스)

```java
// 익명 클래스의 인스턴스를 함수 객체로 사용 - 오래된 방법..
Collections.sort(words, new Comparator<String>() {
  public int compare(String s1, String s2) {
    return Integer.compare(s1.length(), s2.length());
  }
});
```

- 과거 객체 지향 디자인 패턴에는 익명 클래스면 충분했으나, 시간이 지남에 따라 익명 클래스 방식은 코드가 너무 길기 때문에 함수형 프로그래밍에 적합하지 않게됨

- 자바8 부터 ***추상 메서드 하나짜리 인터페이스는 특별한 의미를 인정받아 특별한 대우를 받게 되었다***

  ⇢ 함수형 인터페이스 (+ 추가로 람다로 이용 가능하게 됨)

<br>

##### 람다식으로 리팩토링

```java
// 람다식을 함수 객체로 사용 - 익명 클래스 대체
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

- 타입 추론
  - 람다, 매개변수(s1, s2), 반환값의 타입을 컴파일러가 문맥을 이용해 타입을 추론한다
  - 상황에 따라 타입을 추론하지 못 할경우, 직접 명시해야 한다
  - 타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개면수 타입은 생략하자

<br>

##### 또 다른 리팩토링..

```java
// 비교자 생성 메서드를 이용한 동일한 동작의 코드
Collections.sort(words, comparingInt(String::length));

// 자바8 List 인터페이스에 추가된 sort 메서드를 이용한 동일한 동작의 코드
words.sort(comparingInt(String::length));
```

<br>

<br>

### 람다식 사용의 예

##### 아이템 34의 Operation 열거 타입

```java
// 상수별 클래스 본문과 데이터를 사용한 열거 타입
public enum Operation {
  PLUS("+") {
    public double apply(double x, double y) { return x + y; }
  },
  MINUS("-") {
    public double apply(double x, double y) { return x - y; }
  },
  TIMES("*") {
    public double apply(double x, double y) { return x * y; }
  },
  DIVIDE("/") {
    public double apply(double x, double y) { return x / y; }
  };
  
  private final String symbol;
  
  Operation(String symbol) { this.symbol = symbol; }
  
  @Override
  public String toString() { return symbol; }
  public abstract double apply(double x, double y);
}
```

<br>

##### 람다를 이용한 리팩토링

```java
// 함수 객체(람다)를 인스턴스 필드에 저장해 상수별 동작을 구현한 열거 타입
public enum Operation {
  PLUS("+", (x, y) -> x + y),
  MINUS("-", (x, y) -> x - y),
  TIMES("*", (x, y) -> x * y),
  DIVIDE("/", (x, y) -> x / y);
  
  private final String symbol;
  private final DoubleBinaryOperator op;
  
  Operation(String symbol, DoubleBinaryOperator op) { 
    this.symbol = symbol; 
    this.op = op;  // 함수형 인터페이스 인스턴스를 필드에 저장
  }
  
  @Override
  public String toString() { return symbol; }
  
  public double apply(double x, double y) {
    return op.applyAsDouble(x, y);  // 함수형 인터페이스 인스턴스의 메서드를 호출하여 결과 전달
  }
}
```

- DoubleBinaryOperator 인터페이스는 `java.util.function` 패키지가 제공하는 함수형 인터페이스
- 위의 예제에서는 상수별 클래스 본문이 사용되지 않았지만, 

<br>

##### 람다의 단점

- 코드 줄 수가 많아지면 가독성이 심하게 나빠진다

  ⇢ 이름이 없고 문서화도 못하기 때문에 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 가독성이 심하게 나빠진다

##### 람다식의 장점

- 코드가 간결해진다

  ⇢ 보일러 플레이트 코드가 사라지고 어떤 동작을 하는지가 명확하게 드러난다

<br>

### 익명 클래스와의 차이점

- 추상 클래스의 인스턴스 생성
  - 익명 클래스 : 사용 가능
  - 람다 : 사용 불가(함수형 인터페이스에서만 쓰인다)
- `this` 키워드의 의미
  - 익명 클래스 : 익명 클래스 자신을 가리킴
  - 람다 : 자신을 참조할 수 없음(람다에서의 `this` 는 바깥 인스턴스를 가리킨다)

<br>

### 익명 클래스와 유사점

- 직렬화하는 일은 극히 삼가야 한다
  - 직렬화해야만 하는 함수 객체가 있다면 private 정적 중첩 클래스(독립적으로 동작, [아이템 24](Item24.md))의 인스턴스를 사용하자

<br>

<br>

### 저자의 제안

- 람다는 한줄일 때 가장 좋다
- 길어도 세줄 안에 끝내는게 좋다, 세줄이 넘어가면 가독성이 심하게 나빠진다
- 람다가 길거나 읽기 어렵다면 더 간단히 줄여보거나 람다를 쓰지 않는 쪽으로 리팩토링하라
- 익명 클래스는 함수형 인터페이스가 아닌 타입의 인스턴스를 만들 때만 사용하라

<br>