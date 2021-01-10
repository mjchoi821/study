## Item 22. 인터페이스는 타입을 정의하는 용도로만 사용하라

<br>

### 인터페이스의 용도

인터페이스를 구현하는 클래스가 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에 얘기해주는 것 (타입으로서의 용도)

<br>

### 인터페이스의 용도를 벗어나게 하는 상수 인터페이스

위 용도를 벗어나는 예로 `상수 인터페이스` 가 존재하는데 메서드 없이, `static final` 필드로만 이루어진 인터페이스를 말한다.

```java
public interface Constants {
  double PI = 3.14159;
  double PLANCK_CONSTANT = 6.62606896e-34;
}
```

이는 인터페이스이기에 클래스에서 얼마든지 구현해 사용될 수 있다.

<br>

#### 상수 인터페이스의 문제

```java
// 의도하지 않게 일어나는 일 ver.1
public class Calculations implements Constants {
  public double getReducedPlanckConstant() {
		return PLANCK_CONSTANT / (2 * PI);
	}
}
```

<br>

```java
// 의도하지 않게 일어나는 일 ver.2
public interface Constants {
  public static final int CONSTANT = 1;
}

public class Class1 implements Constants {
  public static final int CONSTANT = 2;  // 경고나 오류없이 컴파일 된다
  
  public static void main(String args[]) throws Exception {
    System.out.println(CONSTANT); // 2가 출력된다
  }
}
```

[참고: wiki-Constant interface](https://en.wikipedia.org/wiki/Constant_interface#cite_note-2)

<br>

이렇듯 상수 인터페이스는 ***안티패턴*** 으로 사용해서는 안되는 방법이다.

위의 안티패턴의 대안으로 아래와 같은 방법들이 있다.

- 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가하는 방법
  - ex. Integer와 Double 에 선언된 MIN_VALUE와 MAX_VALUE 상수
- 열거 타입이 적합한 상수라면 열거 타입 사용 ([아이템 34](Item34.md))
- 인스턴스화 할 수 없는 유틸리티 클래스 사용 ([아이템 4](Item04.md)) 


<br>

#### 상수 인터페이스 개선

```java
// 인스턴스화 할 수 없는 클래스로 만들기
public final class Constants {
  private Constants() {
    // 인스턴스화 방지
  }
  
  public static final double PI = 3.14159;
  public static final double PLANCK_CONSTANT = 6.62606896e-34;
}
```

<br>

```java
import static Constants.PLANCK_CONSTANT;
import static Constants.PI;
// 또는 import static Constants.*; 이렇게도 사용이 가능하다 - static import 사용

public class Calculations {
  public double getReducedPlanckConstant() {
    return PLANCK_CONSTANT / (2 * PI);
  }
}
```

<br>

<br>

***인터페이스는 타입을 정의하는 용도로만 사용해야 한다.***

***상수 공개용 수단으로 사용하지 말자.***

