## Item 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라



유연성과 재사용성, 테스트 용이성을 위해 의존 객체 주입을 사용하라.

어느 클래스가 하나 이상의 자원에 의존할 경우, 생성자에 필요한 자원을 넘겨 받아 사용하도록 하자. 

이는 의존 객체 주입의 한 형태로, 책에서는 맞춤법 검사기를 예로 들고 있다.



```java
public class SpellChecker {
  private final Lexion dictionary;
  
  public SpellChecker(Lexion dictionary) {
    this.dictionary = Objects.requireNonNull(dictionary);
  }
  
  public boolean isValid(String word) {
    // ...
  }
  
  public List<String> suggestions(String typo) {
    // ...
  }
}
```



사전을 외부에서 주입받음으로써 언어별로 다른 사전을 받을 수도 있고, 특수 어휘용 사전과 같은 특수한 사전도 외부에서 주입받아 동일하게 사용할 수 있다.

또한 불변(아이템 17)을 보장하여 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있기도 하다. 의존 객체 주입은 생성자, 정적 팩토리(아이템 1), 빌더(아이템 2) 모두에 똑같이 응용할 수 있다.



이 패턴의 변형으로, 생성자에 자원 팩토리를 넘겨주는 방식이 있다. 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체이며, 팩토리 메서드 패턴이라 한다.

자바 8에서 소개한 supplier<T> 인터페이스가 팩토리를 표현한 완벽한 예다. 

```java
Mosaic create(Supplier<? extends Tile> tileFactory) {
  // ...
}
```



위 코드는 클라이언트가 제공한 팩토리가 생성한 Tile 들로 구성된 Mosaic 를 만드는 메서드다.



의존 객체 주입이 유연성과 테스트 용이성을 개선해주긴 하지만, 큰 프로젝트와 같이 의존성이 많아지게되면 복잡성이 늘어나고 관리도 어려워지게 된다. 

이 때 대거(Dagger), 주스(Guice), 스프링(Spring) 같은 의존 객체 주입 프레임워크를 사용하면 이런 어려움을 해소할 수 있다.