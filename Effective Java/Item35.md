## Item 35. ordinal 메서드 대신 인스턴스 필드를 사용하라

<br>

모든 열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 ordinal 이라는 메서드를 제공한다.

<br>

**ordinal 을 잘못 사용한 예** - 따라하지 말 것!

```java
public enum Ensemble {
  SOLO, DUET, TRIO, QUARTET, QUINTET, SEXTET, SEPTET, OCTET, NONET, DECTET;
  
  public int numberOfMusicians() { return ordinal() + 1; }
}
```

위 예제는 동작하지만 유지보수하기가 끔찍한 코드다.

상수 선언 순서를 바꾸는 순간 numberOfMusicians 가 오동작하며, 이미 사용중인 정수와 값이 같은 상수는 추가할 방법이 없다.

또한 값을 중간에 비워둘 수도 없다.(+1 로 값을 증가시키므로) 이런 이유로 코드가 깔끔하지 못할 뿐 아니라, 쓰이지 않는 값이 많아질수록 실용성이 떨어진다.

<br>

해결방법은 열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 않고, 인스턴스 필드에 저장하여 사용하는 것이다.

```java
public enum Ensemble {
  SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5), 
  SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8), 
  NONET(9), DECTET(10), TRIPLE_QUARTET(12);
  
  private final int numberOfMusicians;
  Ensemble(int size) { this.numberOfMusicians = size; }
  public int numberOfMusicians() { return numberOfMusicians; }
}
```

<br>

Enum 의 API 문서를 보면 ordinal 에 대해 이렇게 쓰여 있다고 한다. 

> 대부분의 프로그래머는 이 메서드를 쓸 일이 없다. 
>
> 이 메서드는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다.

따라서 이런 용도가 아니라면 ordinal 메서드는 사용하지 말자.

<br>