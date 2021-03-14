# Item 65. 리플렉션보다는 인터페이스를 사용하라

<br>

### 리플렉션?

프로그램에서 임의의 클래스에 접근 할 수 있다.

- Class 객체 ⇢ 생성자, 메서드, 필드에 해당하는 Constructor, Method, Field 인스턴스를 가져올 수 있다

  가져온 인스턴스를 이용해 각각에 연결된 실제 생성자, 메서드, 필드를 조작할 수도 있다

- 인스턴스 ⇢ 클래스의 멤버 이름, 필드 타입, 메서드 시그니처 등을 가져올 수 있다 

<br>

### 단점

- 컴파일타임 타입 검사가 주는 이점을 누릴 수 없다.

  주의해서 대비 코드를 작성해두지 않았다면 런타임 오류가 발생한다.

- 코드가 지저분하고 장황해진다.

  읽기 어렵다.

- 성능이 떨어진다.

  일반 메서드 호출보다 훨씬 느리다.

리플렉션을 써야하는 코드 분석 도구나 의존관계 주입 프레임워크들 마저 리플렉션 사용을 점차 줄이고 있다.

<br>

리플렉션은 아주 제한된 형태로만 사용해야 단점을 피하고 이점만 취할 수 있다.

컴파일 타임에 이용할 수 없는 클래스를 사용해야만 하는 프로그램은 비록 컴파일타임이라도 적절한 인터페이스나 상위 클래스를 이용할 수는 있을 것이다.([아이템64](Item64.md)) 다행히 이런 경우라면 리플렉션은 인스턴스 생성에만 쓰고 이렇게 만든 인스턴스는 인터페이스나 상위 클래스로 참조해 사용하자.

<br>

**리플렉션으로 생성하고 인터페이스로 참조해 활용하는 예제**

```java
public static void main(String[] args) {
  Class<? extends Set<String>> cl = null;
  try {
    cl = (Class<? extends Set<String>) Class.forName(args[0]); // 비검사 형변환
  } catch (ClassNotFoundException e) {
    fatalError("클래스를 찾을 수 없습니다.");
  }
  
  // 생성자를 얻는다
  Constructor<? extends Set<String>> cons = null;
  try {
    cons = cl.getDeclaredConstructor();
  } catch (NoSuchMethodException e) {
    fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
  }
  
  // 집합의 인스턴스를 만든다
  Set<String> s = null;
  try {
    s = cons.newInstance();
  } catch (IllegalAccessException e) {
    fatalError("생성자에 접근할 수 없습니다.");
  } catch (InstantiationException e) {
    fatalError("클래스를 인스턴스화할 수 없습니다.");
  } catch (InvocationTargetException e) {
    fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
  } catch (ClassCastException e) {
    fatalError("Set을 구현하지 않은 클래스입니다.");
  }
  
  // 생성한 집합을 사용한다
  s.addAll(Arrays.asList(args).subList(1, args.length));
  System.out.println(s);
}

private static void fatalError(String msg) {
  System.err.println(msg);
  System.exit(1);
}
```

간단한 프로그램이지만 여기서 선보인 기법은 꽤나 강력하며, 이 프로그램은 손쉽게 제네릭 집합 테스터로 변신할 수 있다. 

`Set` 구현체를 공격적으로 조작해보며 `Set` 규약을 잘 지키는지 검사해볼 수 있다. 비슷하게, 제네릭 집합 성능 분석 도구로 활용할 수도 있다.

이 기법은 완벽한 서비스 제공자 프레임워크([아이템 1](Item01.md)) 를 구현할 수 있을 만큼 강력하다. 대부분의 경우 리플렉션 기능은 이정도만 사용해도 충분하다.

<br>

### 예제에서 볼 수 있는 리플렉션의 단점

1. 런타임에 총 여섯가지나 되는 예외를 던질 수 있다

   특히 모든 예외는 인스턴스를 리플렉션 없이 생성했다면 컴파일타임에 잡아낼 수 있었을 예외들이다.

2. 클래스 이름만으로 인스턴스를 생성해내기 위해 무려 25줄이나 되는 코드를 작성했다

   리플렉션이 아니라면 생성자 호출 한 줄로 끝났을 일이다.

<br>

<br>

### 핵심 정리

리플렉션은 복잡한 특수 시스템을 개발할 때 필요한 강력한 기능이지만, 단점도 많다.

되도록 객체 생성에만 사용하고, 생성한 객체를 이용할 때는 적절한 인터페이스나 컴파일타임에 알 수 있는 상위 클래스로 형변환해 사용해야 한다.

<br>

