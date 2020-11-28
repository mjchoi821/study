## Item 1. 생성자 대신 정적 팩토리 메서드를 고려하라

Item1은 2장의 객체 생성과 파괴에 포함된 내용으로 static factory method 가 constructor 보다 좋은 장점과 단점을 아래와 같이 설명하고 있다.



### 정적 팩토리 메서드란?

객체 생성 역할을 하는 정적 메서드를 말한다.



### 장점

#### 1. 이름을 가질 수 있다.

생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못한다. 반면 정적 팩토리 메서드는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.

- 사용 예

```java
User user1 = new User(); // User 인스턴스를 생성하지만 어떤 값을 가지고 있는건지 아닌건지 알 수 없다
User user2 = User.emptyUser(); // 아무런 내부 값을 가지지 않는 User 인스턴스를 리턴할 것이다

User user3 = new User("김철수"); // 인자로 전달한 값이 설정되어있는 User 인스턴스를 생성하지만 인자값이 어느 변수에 설정되는지 알 수 없다
User user4 = User.getUserWithName("김철수"); // 인자로 전달한 값을 name 으로 설정한 User 인스턴스를 리턴할 것이다
```

<br />

하나의 시그니처로는 생성자를 하나만 만들 수 있다. 하지만 이름을 가질 수 있는 정적 팩토리 메서드에는 이런 제약이 없으므로 유용하게 쓰일 수 있다.
한 클래스에 시그니처가 같은 생성자가 여러 개 필요할 것 같으면, 생성자를 정적 팩토리 메서드로 바꾸고 각각의 차이를 잘 드러내는 이름을 지어주자.

- 사용 예

```java
public class StaticFactoryMethod {
    private String name;
    private String value;

    // 생성자 - 파라미터가 아무것도 없을 수 있고
    public StaticFactoryMethod() {
    }

    // 하나의 String 타입의 파라미터를 받을 수도 있다.
    public StaticFactoryMethod(String name) {
        this.name = name;
    }

    // 그러나 생성자는 하나의 String 타입의 파라미터를 받고 인스턴스를 리턴하는 생성자를
    // 단 하나 밖에 갖을 수 없다. (== 시그니처가 같은 생성자를 만들 수 없는 한계가 있다)
    // 만약 이렇게 동일한 시그니처를 갖고 인스턴스를 리턴해야하는 경우라면, 생성자 대신 정적 팩토리 메서드를 이용하라

    // 아래 메서드는 위의 StaticFactoryMethod(String name) 과 시그니처가 동일하기 때문에 사용할 수 없다.
//    public StaticFactoryMethod(String value) {
//        this.value = value;
//    }

    // 이런 경우, 정적 팩토리 메서드를 이용하면 두가지 모두 사용이 가능해진다.
    public static StaticFactoryMethod getInstaceWithName(String name) {
        return new StaticFactoryMethod(name);
    }

    // 이렇게 정적 팩토리 메서드는 파라미터와 리턴타입을 동일하게 사용할 수 있다.
    public static StaticFactoryMethod getInstaceWithValue(String value) {
        StaticFactoryMethod instance = new StaticFactoryMethod();
        instance.value = value;
        return instance;
    }
}
```

<br />

#### 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.

생성자는 호출 시 무조건 인스턴스를 만들어 반환하도록 만들어진 메서드이다. 반면 정적 팩토리 메서드를 사용하면, 인스턴스를 통제 할 수 있다.

불변 클래스(immutable class)는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다. 특히 생성 비용이 큰 객체가 자주 요청되는 상황이라면 성능을 상당히 끌어올려준다. Flyweight 패턴도 이와 비슷한 기법이라 할 수 있다.

반복되는 요청에 같은 객체를 반환하는 식으로 정적 팩토리 방식의 클래스는 언제 어느 인스턴스를 살아있게 할지를 통제할 수 있다.

이런 클래스를 인스턴스 통제(instance-controlled) 클래스라 한다. 인스턴스를 통제하여 클래스를 싱글턴으로 만들수도 인스턴스화 불가(noninstantiable)로 만들수도 있다. 인스턴스 통제는 Flyweight 패턴의 근간이 되며, 열거 타입은 인스턴스가 하나만 만들어짐을 보장한다.



-  사용 예 - public static method 방식의 싱글턴

```java
// SomeClass 의 인스턴스는 내부에서밖에 생성할 수 없고(통제), getInstance 함수를 통해 외부에 공유되는데,
// 현재 내부에서 단 한번의 new 를 통해 변수에 할당해 사용하고 있기때문에, 하나의 인스턴스만으로 사용하고 있다는걸 알 수 있다.
public class SomeClass {
  private static final SomeClass INSTANCE = new SomeClass();
  
  private SomeClass() {}
  
  public static SomeClass getInstance() {
    return INSTANCE;
  }
  
  // ...
}
```



주의해야 할 점은 인스턴스를 공유해서 사용할 경우, 공유된 인스턴스의 값이 변경가능할 때 여파가 여러곳에 미치게되므로, 이를 유의해서 사용해야 할 것이다.



> **Flyweight 패턴** (Flyweight == 플라이급, 권투에서 체급이 가장 가벼운급) <p>
> 가능한한 인스턴스를 공유해 new 연산자를 통한 메모리 사용을 최소화하는 방법


<br />

#### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

생성자는 호출 시 해당 클래스의 인스턴스를 반환한다. 하지만 정적 팩토리 메서드를 사용한다면, 메서드 내부에서 현재의 클래스 뿐만아니라 하위 타입의 클래스(현재 클래스를 상속한)도 반환이 가능해진다.

이는 반환할 객체의 클래스를 자유롭게 선택할 수 있게하는 ***유연성*** 을 의미한다.

API 를 만들 때 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API 를 작게 유지할 수 있다. 이는 인터페이스를 정적 팩토리 메서드의 반환 타입으로 사용하는 인터페이스 기반 프레임워크(아이템 20)를 만드는 핵심 기술이기도 하다.

자바 8전에는 인터페이스에 정적 메서드를 선언할 수 없었기에 정적 메서드가 필요한경우 인스턴스화를 할 수 없는 정적 메서드만이 존재하는 동반 클래스(companion class)를 만들어 사용했다. 대표적인 예로 자바의 컬렉션 프레임워크(java.util.Collections)를 들수있다.

자바 8부터는 인터페이스가 정적 메서드를 가질 수 있게되어 인스턴스화 불가인 동반 클래스를 둘 이유가 별로 없어졌지만, 구현에 따라서 충분히 쓰일 수 있는 방식이기에 기억해두자.


<br />

#### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

입력 매개변수에 따라 내부적으로 다른 클래스(하위 타입인)를 반환할 수 있다. 물론 3번에서 이야기한 것과 같이 하위타입일 경우만 가능하다.

'내부 구현에 외부가 영향을 받지 않을 수 있다' 로 이해해도 무방한 이야기이다.

EnumSet 클래스를 예로들어 설명하는데, 이 클래스는 정적 팩토리 메서드만 지원하고 내부적으로 원소의 수에 따라 두 가지의 하위 클래스 중 하나의 인스턴스를 반환한다. 사용하는 입장에서는 EnumSet 의 하위 클래스이기만 하면 문제없이 동작하게 되어있으므로, 유연함과 캡슐화의 장점을 가진다고 말할 수 있겠다.


<br />

#### 5. 정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

유연성에 대한 이야기로, 동적으로 인스턴스를 반환하는 것에 대한 이야기이다.

책에서는 서비스 제공자 프레임워크(service provider framework)의 예로 JDBC 와 DI(Dependency Injection, 의존성 주입) 를 이야기한다.

자바 6부터는 java.util.ServiceLoader 라는 범용 서비스 제공자 프레임워크가 제공되어 프레임워크를 직접 만들 필요가 거의 없어졌다고 한다. 자세한 내용은 아이템 59에서 다룰 예정이다.


<br /><br />

### 단점

#### 1. 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 없다.

상속을 하려면 public 이나 protected 생성자가 필요하다. 따라서 이들 생성자가 없이 정적 팩토리 메서드만을 제공한다면 하위 클래스를 생성 할 수 없다.

상속을 이해하고 완벽하게 문제없이 만들 수 있지 않는 이상, 상속보다는 합성을 이용하는것이 좋다.

따라서 이 단점은 단점이 아니라 오히려 장점이 될 수 있는 내용이기도 하다.


<br />

#### 2. 정적 팩토리 메서드는 프로그래머가 찾기 어렵다.

생성에 관여하고 있다는 사실이 API 문서등에 명확히 표현되지 않아서, 이미 사용되고 있는 명명규칙에 의존해 표현하거나 따로 API 문서를 작성해 안내해야 한다.

언젠가 javadoc 이 알아서 생성에 관여하는 정적 팩토리 메서드를 잘 안내해줬으면 하는 저자의 바람이 인상깊었다.



<br />

### 정적 팩토리 메서드 네이밍 컨벤션

- `from` : 하나의 매개변수를 받아 해당 타입의 인스턴스를 반환

  ```java
  Date d = Date.from(instance);
  ```

  

- `of` : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환

  ```java
  Set<Color> blackAndWhite = EnumSet.of(Color.BLACK, Color.WHITE);
  ```

  

- `valueOf` : from 과 of 보다 상세한 버전

  ```java
  BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
  ```

  

- `instance` or `getInstance` : 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않음

  ```java
  StackWalker luke = StackWalker.getInstance(options);
  ```

  

- `create` or `newInstance` : instance 혹은 getInstance 와 같지만, 매번 새로운 인스턴스를 생성해 반환

  ```java
  Object newArray = Array.newInstance(classObject, arrayLen);
  ```

  

- `get{Type}` : getInstance 와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메서드를 정의할 때 사용

  ```java
  FileStore fs = Files.getFileStore(path);
  ```

  

- `new{Type}` : newInstance 와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메서드를 정의할 때 사용

  ```java
  BufferedReader br = Files.newBufferedReader(path);
  ```

  

- `{type}` : get 이나 new 를 모두 제외한 {type}만 작성하는 방식

  ```java
  List<Message> messages = Collections.list(otherTypeMessages);
  ```

  



지금까지 정적 팩토리 메서드의 장단점을 알아봤다.

앞으로 인스턴스를 생성할 때 무조건 public 생성자로 만들기보다, 정적 팩토리 메서드가 더 적절하진 않은지 고민해 보자.
