## Item 10. equals는 일반 규약을 지켜 재정의하라



equals 메서드는 재정의하기 쉬워 보이지만 자칫하면 끔찍한 결과를 초래한다.
이런 ***문제를 일으키지 않는 가장 쉬운 방법은 아예 재정의하지 않는 것*** 이다.



아래와 같은 상황 중 하나에 해당한다면 재정의하지 않는 것이 최선이다.

- 각 인스턴스가 본질적으로 고유하다
  ex) Thread, 모든 쓰레드는 고유하다

- 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다
  ex) 같은 정규표현식인지를 검사하는게 아니라면, 기본 Object 의 equals 만으로 충분하다

- 상위 클래스에서 재정의한 equals 가 하위 클래스에도 딱 들어맞는다
  ex) Set/List/Map 구현체들은 각각의 Abstract{Set/List/Map} 이 구현한 equals 를 상속받아서 쓴다

- 클래스가 private이거나 package-private 이고 equals 메서드를 호출할 일이 없고, 위험을 철저히 회피하고 싶다면 equals 를 오버라이딩해 예외를 넣어두자

  ```java
  @Override public boolean equals(Object o) {
    throw new AssertionError(); // 호출 금지!
  }
  ```



### equals 를 재정의 해야 할 때?

인스턴스의 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals 가 논리적 동치성을 비교하도록 재정의되지 않았을 때이다.
주로 값 클래스들이 여기 해당한다. Integer 와 String 같은 클래스를 말하며 두 값 객체를 equals 로 비교할 때 프로그래머는 객체가 같은지가 아니라 값이 같은지를 알고 싶어 할 것이다. equals 가 논리적 동치성을 확인하도록 재정의하면, 그 인스턴스는 값을 비교하길 원하는 프로그래머의 기대에 부응함은 물로 Map의 키와 Set의 원소로 사용할 수 있게 된다.



### equals 메서드 재정의 시 반드시 따라야 하는 일반 규약

> equals 메서드는 동치관계(equivalence relation)를 구현하며, 다음을 만족한다.

- 반사성(reflexivity) : null이 아닌 모든 참조 값 x에 대해, `x.equals(x)` 는 true다.
- 대칭성(symmetry) : null이 아닌 모든 참조 값 x, y에 대해, `x.equals(y)` 가 true면 `y.equals(x)` 도 true다.
- 추이성(transitivity) : null이 아닌 모든 참조 값 x, y, z에 대해, `x.equals(y)` true이고 `y.equals(z)` 도 true면 `x.equals(z)` 도 true다.
- 일관성(consistency) : null이 아닌 모든 참조 값 x, y에 대해, `x.equals(y)` 를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
- null-아님 : null 이 아닌 모든 참조 값 x에 대해, `x.equals(null)` 은 false다.



꼭 필요한 경우가 아니면 equals 를 재정의하지 말자. 많은 경우에 Object의 equals가 이미 잘 동작하고 있다.
그럼에도 불구하고 재정의해야 한다면, 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯가지 규약을 확실히 지켜가며 비교해야 한다.