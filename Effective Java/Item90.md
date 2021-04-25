# Item 90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

<br>

### 직렬화 프록시 패턴(serialization proxy pattern)?

#### 생성 방법

- 단 하나의 생성자를 가진 중첩 클래스를 `private static` 으로 선언한다

- 생성자는 바깥 클래스를 매개변수로 받아야 하며, 인수로 넘어온 인스턴스의 데이터를 복사한다

  (일관성 검사나 방어적 복사가 없어도 된다)

- 바깥 클래스와 직렬화 프록시 모두 `Serializable` 을 구현한다고 선언해야 한다

- 바깥 클래스에 `writeReplace` 메서드를 추가한다

  ```java
  // 직렬화 프록시 패턴의 범용적인 메서드
  private Object writeReplace() {
    return new SerializationProxy(this);
  }
  ```

  - `writeReplace` 메서드는 직렬화가 이뤄지기 전에 바깥 클래스의 인스턴스를 직렬화 프록시로 변환해준다

- 불변식을 훼손 공격을 막아내는 `readObject` 메서드를 추가한다

  ```java
  // 직렬화 프록시 패턴용 readObject 메서드
  private void readObject(ObjectInputStream stream) throws InvalidObjectException {
    throw new InvalidObjectException("프록시가 필요합니다.");
  }
  ```

- 바깥 클래스와 논리적으로 동일한 인스턴스를 반환하는 `readResolve` 메서드를 SerializationProxy 클래스에 추가한다

  ```java
  private Object readResolve() {
    return new Period(start, end); // public 생성자를 사용
  }
  ```

  - 공개된 API 만을 사용해 바깥 클래스의 인스턴스를 생성한다

    > ⇢ 직렬화 프록시 패턴의 포인트!
    > 직렬화는 생성자를 이용하지 않고도 인스턴스를 생성하는 기능을 제공하는데, 이 패턴은 직렬화의 이런 언어도단적 특성을 상당 부분 제거한다. 일바 인스턴스를 만들 때와 똑같은 생성자, 정적 팩토리, 혹은 다른 메서드를 사용해 역직렬화된 인스턴스를 생성하는 것이다.

<br>

### 직렬화 프록시 패턴의 장점

- 가짜 바이트 스트림 공격과 내부 필드 탈취 공격을 프록시 수준에서 차단해준다
- 직렬화 프록시는 필드를 `final` 로 선언해도 되므로 원하는 클래스를 불변([아이템 17](Item17.md))으로 만들수도 있다
- 유효성 검사나 공격에 대한 고민을 하지 않아도 된다
- 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동한다 (EnumSet 참고)

### 직렬화 프록시 패턴의 한계

- 클라이언트가 멋대로 확장할 수 있는 클래스([아이템 19](Item19.md))에는 적용할 수 없다

- 객체 그래프에 순환이 있는 클래스에도 적용할 수 없다

### 직렬화 프록시 패턴의 단점

- 느린 속도
  - 저자의 PC 에서는 방어적 복사보다 14% 의 느린속도를 보였다고 한다

<br>

### 핵심 정리

- 제 3자가 확장할 수 없는 클래스라면 가능한 한 직렬화 프록시 패턴을 사용하자
- 이 패턴이 중요한 불변식을 안정적으로 직렬화해주는 가장 쉬운 방법일 것이다