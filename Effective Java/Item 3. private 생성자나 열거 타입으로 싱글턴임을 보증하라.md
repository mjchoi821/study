## Item 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라



### 싱글턴(singleton) 이란

인스턴스를 오직 하나만 생성할 수 있는 클래스



#### 사용되는 곳

- 함수와 같은 무상태(stateless) 객체
- 설계상 유일해야 하는 시스템 컴포넌트



#### 단점

- 테스트하기 어려워 질 수 있음
  (인터페이스로 정의된 싱글턴이 아니라면 해당 싱글턴 인스턴스를 mock 으로 대체할 수 없기 때문에)



### 싱글턴을 만드는 방식

#### 1. public static final 필드 방식

- 생성자는 private 으로 감추어, 외부에서 인스턴스를 생성할 수 없게 한다
- 사용 예

```java
public class Exam1 {
  public static final Exam1 INSTANCE = new Exam1(); // 외부에서는 Exam1.INSTANCE 로 접근해 사용한다
  
  private Exam1() {} // 외부에서 생성자를 호출 할 수 없다. 다만, 리플렉션 API 로 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있다. 이 예외를 방지하기 위해서는 생성자를 수정하여 두번째 객체가 생성되려 할 때 예외를 던지게 처리하면 된다.
  
  // ...
}
```



#### 2. static factory method 방식

- 1과 마찬가지로 생성자는 private 으로 감추어, 외부에서 인스턴스를 생성할 수 없게 한다

  리플렉션 API 예외도 동일하게 적용된다

- 사용 예

```java
public class Exam2 {
  private static final Exam2 INSTANCE = new Exam2(); // private 으로 내부에서만 접근이 가능한 멤버
  
  private Exam2() {} // 외부에서 생성자를 호출 할 수 없다, 리플렉션을 통한 예외는 1번 방식과 동일하게 적용된다
  
  // 외부에서는 Exam2.getInstance() 로 접근해 사용한다
  public static Exam2 getInstance() {
    return INSTANCE;
  }
}
```



#### 3. 열거(enum) 타입 방식

- public 필드 방식과 비슷하지만 더 간결하고, 추가 노력없이 serialize 가 가능하고, 리플렉션 예외도 생기지 않고, 제 2의 인스턴스가 생성되지도 않는다 (컴파일 타임에 모든것이 정해진다고 보면된다)

- 사용 예

```java
public enum Exam3 {
  INSTANCE;
  
  public void someMethod() {
    // do something
  }
}
```





### 주의할 점

싱글턴 패턴을 Anti-Pattern 으로 보기도 할만큼 지양해야 한다는 이야기도 있다.

싱글턴을 안티 패턴으로 보는 이유는 전역 상태를 조장하고, 그 때문에 설계상의 문제점을 찾기 힘들게하며 테스트하기 어렵게 만들기 때문이다.

클래스사이에 결합도가 높아지게 되어, 수정 시 많은곳에 영향을 미칠수도 있다.

이렇듯 싱글턴에 대해서 주의깊게 알고 필요한 곳에서 적절하게 사용해야 하겠다.



[참고 - 위키피디아](https://en.wikipedia.org/wiki/Singleton_pattern)





