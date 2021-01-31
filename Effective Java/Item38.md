## Item 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

<br>

열거 타입은 확장 할 수 없지만(상속), 임의의 인터페이스를 구현할 수 있다.

이를 이용해 다음과 같은 확장할 수 있는 열거 타입을 만들어 **사용자 확장 연산** 과 같은 기능을 추가한 열거 타입을 만들 수 있다.

```java
// 아이템 34의 Operation 타입을 확장할 수 있게 만든 코드
public interface Operation {
  double apply(double x, double y);
}

public enum BasicOperation implements Operation {
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
  BasicOperation(String symbol) {
    this.symbol = symbol;
  }
  
  @Override
  public String toString() {
    return symbol;
  }
}
```

- 열거 타입인 BasicOperation 은 확장할 수 없지만 인터페이스인 Operation 은 확장할 수 있고, 이 인터페이스를 연산의 타입으로 사용하면 된다
- 이렇게 하면 Operation 을 구현한 또 다른 열거 타입을 정의해 기본 타입인 BasicOperation 을 대체할 수 있다

<br>

#### Operation 을 확장해보자

```java
public enum ExtendedOperation implements Operation {
  EXP("^") {
    public double apply(double x, double y) { return Math.pow(x, y); }
  }, 
  REMAINDER("%") {
    public double apply(double x, double y) { return x % y; }
  };
  
  private final String symbol;
  ExtendedOperation(String symbol) {
    this.symbol = symbol;
  }
  
  @Override
  public String toString() {
    return symbol;
  }
}
```

- 확장한 ExtendedOperation 은 기존 연산을 쓰던 곳이면 어디든 쓸 수 있다 (인터페이스 기준)

<br>

#### 인터페이스를 이용한 확장 enum 의 사용

Operation enum 의 테스트를 위한 메서드를 만들어 보자.

1. 한정적 타입 토큰(class 리터럴)을 이용한 방식 ([아이템 33](Item33.md))

```java
private static <T extends Enum<T> & Operation> void test(
  	Class<T> opEnumType, double x, double y) {
  for (Operation op: opEnumType.getEnumConstants())
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}

// 코드 사용
public static void main(String[] args) {
  double x = Double.parseDouble(args[0]);
  double y = Double.parseDouble(args[1]);
  test(ExtendedOperation.class, x, y);
}
```

- class 리터럴을 넘겨 확장된 연산과 기본 연산 모두를 사용 할 수 있다
- 제네릭 메서드의 선언(`<T extends Enum<T> & Operation>`)의 내용은 열거 타입인 동시에 Operation 의 하위 타입이어야 한다는 의미이다

<br>

2. 한정적 와일드카드 타입을 이용한 방식 ([아이템 31](Item31.md))

```java
private static void test(Collection<? extends Operation> opSet, double x, double y) {
  for (Operation op: opSet)
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}

// 코드 사용
public static void main(String[] args) {
  double x = Double.parseDouble(args[0]);
  double y = Double.parseDouble(args[1]);
  test(Arrays.asList(ExtendedOperation.values()), x, y);
} 
```

- 1번보다 덜 복잡하다
- 여러 구현 타입의 연산을 조합해 넘길 수 있기 때문에 1번 보다 조금 더 유연해졌다고 볼 수 있다
- 반면 특정 연산에서는 EnumSet 과 EnumMap 을 사용하지 못한다

<br>

인터페이스를 이용해 확장 가능한 열거 타입을 흉내내는 방식의 한 가지 사소한 문제점은 **열거 타입끼리 구현을 상속할 수 없다** 는 점이다.

현재의 예제 코드는 코드에서 연산기호를 저장하고 찾는 로직이 모든 부분에 들어가야만 한다. 다행히 이 예제는 코드의 중복량이 적어 문제되지 않지만, 인터페이스에서 정의하는 기능이 많다면 그 부분을 별도의 도우미 클래스나 static 메서드로 분리하는 방식으로 코드 중복을 없앨 수 있을 것이다.

<br>

자바 라이브러리도 이런 패턴을 사용한다. 

`java.nio.file.LinkOption` 열거 타입은 CopyOption 과 OpenOption 인터페이스를 구현했다.

<br>

<br>

***열거 타입 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다.***

<br>