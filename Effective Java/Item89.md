# Item 89. 인스턴스 수를 통제해야 한다면 readResolve 보다는 열거 타입을 사용하라

<br>

### `readResolve` 메서드 정의 시

`Serializable` 을 구현하는 클래스에서 `readResolve` 메서드 정의 시 역직렬화 후 새로 생성된 객체를 인수로 `readResolve` 메서드가 호출되며 이 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환된다.

**사용 예**

```java
// 인스턴스 통제 가능 (개선의 여지가 있는)
private Object readResolve() {
  return INSTANCE; // 실제 사용될 인스턴스 전달, 먼저 역직렬화한 객체는 무시한다
}
```

#### 인스턴스 통제 목적으로 사용이 적절한가?

`readResolve` 메서드를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 `transient` 로 선언해야 한다. 그렇지 않으면 [아이템88](Item88.md) 에서 본 MutablePeriod 공격과 비슷한 방식으로 `readResolve` 메서드가 수행되기 전에 역직렬화된 객체의 참조를 공격할 여지가 남는다.

`readResolve` 메서드를 사용해 역직렬화된 인스턴스에 접근하지 못하게 하는 방법은 깨지기 쉽고 신경을 많이 써야 하는 작업이기에 필드의 `transient` 선언보다 싱글턴 클래스를 원소 하나짜리 ***열거 타입*** 으로 바꾸는 편이 더 나은 선택이다([아이템 3](Item03.md)).

(열거 타입은 선언한 상수 외에 다른 객체는 존재하지 않음을 자바가 보장해준다. 임의의 네이티브 코드를 수행할 수 있는 특권을 가로챈 공격자에게는 모든 방어가 무력화 된다)

<br>

**잘못 구현된 싱글턴 클래스**

```java
// transient 가 아닌 참조 필드를 가지고 있다..
public class Elvis implements Serializable {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() { }
  
  private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" }
  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }
  
  private Object readResolve() {
    return INSTANCE;
  }
}
```

**위의 클래스를 악용하는 도둑 클래스**

```java
// 이 클래스를 이용해 아이템 88 과 같은 방법으로 악용될 수 있다
public class ElvisStealer implements Serializable {
  static Elvis impersonator;
  private Elvis payload;
  
  private Object readResolve() {
    impresonator = payload;
    
    return new String[] { "A Fool Such as I" };
  }
  private static final long serialVersionUID = 0;
}
```

<br>

**열거 타입 싱글턴**

```java
// 전통적인 싱글턴보다 우수하다
public enum Elvis {
  INSTANCE;
  private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };
  public void printFAvorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }
}
```

<br>

### `readResolve` 메서드의 접근성

- `final` 클래스에서라면 `readResolve` 메서드는 `private` 이어야 한다

- `final` 이 아닌 클래스에서는 다음을 고려해야 한다

  - `private` 으로 선언하면 하위 클래스에서 사용할 수 없다

  - `package-private` 으로 선언하면 같은 패키지에 속한 하위 클래스에서만 사용할 수 있다

  - `protected` 나 `public` 으로 선언하면 이를 재정의하지 않은 모든 하위클래스에서 사용할 수 있다

     `protected` 나 `public` 이면서 하위 클래스에서 재정의하지 않았다면, 하위 클래스의 인스턴스를 역직렬화 시 상위 클래스의 인스턴스를 생성하여 `ClassCastException` 을 일으킬 수 있다

<br>

### 핵심정리

- 불변식을 지키기 위해 인스턴스를 통제해야 한다면 가능한 한 열거 타입을 사용하자
- `readResolve` 메서드를 사용한다면 그 클래스에서 모든 참조 타입 인스턴스 필드를 `transient` 로 선언해야 한다