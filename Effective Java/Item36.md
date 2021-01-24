## Item 36. 비트 필드 대신 EnumSet을 사용하라

<br>

#### 비트 필드(bit field)란?

각 비트마다 서로 다른 상태를 저장하는 것

<br>

<br>

열거한 값들이 집합으로 사용될 경우, 예전에는 아래와 같이 정수 열거 패턴을 사용해 왔다.

**비트 필드 열거 상수** - 예전 방법!

```java
public class Text {
  public static final int STYLE_BOLD      = 1 << 0; // 1
  public static final int STYLE_ITALIC    = 1 << 1; // 2
  public static final int STYLE_UNDERLINE = 1 << 2; // 4
  public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8
  
  // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR 한 값이다
  public void applyStyles(int styles) {
    // ...
  }  
}
```

<br>

**비트 필드 열거 상수 사용**

```java
// 비트별 OR 를 사용해 여러 상수를 하나의 집합으로 모은다
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

비트 필드를 사용하면 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있다.

하지만 비트 필드는 정수 열거 상수의 단점을 그대로 지니며, 추가로 해석하기 어렵고 모든 원소를 순회하기도 까다로운 등의 단점을 지니고 있다.

<br>

이런 불편함에 대한 대안으로 java.util 패키지의 EnumSet 클래스를 사용하라.

EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.

Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 Set 구현체와도 함께 사용할 수 있다.

<br>

**EnumSet - 비트 필드를 대체하는 현대적 기법**

```java
public class Text {
  public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
  
  // 어떤 Set 을 넘겨도 되나, EnumSet이 가장 좋다
  public void applyStyles(Set<Style> styles) { 
    // ... 
  }
}
```

<br>

**EnumSet 사용**

```java
text.applyStyles(EnumSet.of(Style.BOLD, Style, ITALIC));
```

<br>

위의 applyStyles 메서드가 EnumSet<Style> 이 아닌 Set<Style> 을 받은 이유는, 모든 클라이언트가 EnumSet을 건네리라 짐작되는 상황이라도 이왕이면 인터페이스로 받는 게 일반적으로 좋은 습관이기 때문이다([아이템 64]((Item64.md))). 이렇게 하면 특이한 클라이언트가 다른 Set 구현체를 넘기더라도 처리할 수 있게 된다.

<br>

<br>

---

<br>

#### 참고

[Bit Field Java Implementation](https://jarombek.com/blog/jun-23-2018-bit-field#bit-field)

<br>