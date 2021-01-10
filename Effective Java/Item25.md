## Item 25. 톱레벨 클래스는 한 파일에 하나만 담으라

  

소스 파일 하나에 톱레벨 클래스를 여러 개 선언하더라도 자바 컴파일러는 문제삼지 않는다. 

그러나 아무런 득이 없을 뿐더러 심각한 위험을 감수해야 하는 행위임을 알고, 어째서 사용하면 안되는지 알아보자.

  

```java
// Main.java 파일
public class Main {
  public static void main(String[] args) {
    System.out.println(Utensil.NAME + Dessert.NAME);
  }
}
```

  

```java
// Utensil.java 파일
class Utensil {
  static final String NAME = "pan";
}

class Dessert {
  static final String NAME = "cake";
}
```

  

```java
// Dessert.java 파일
class Utensil {
  static final String NAME = "pot";
}

class Dessert {
  static final String NAME = "pie";
}
```

위와 같이 3개의 파일이 존재할 때, 컴파일 순서에 따라 사용되는 클래스가 달라지게 된다.

  

```shell
javac Utensil.java Dessert.java
// 컴파일 오류 발생, duplicate class

javac Main.java Utensil.java
// 컴파일 성공, 결과는 pancake

javac Main.java Dessert.java
// 컴파일 성공, 결과는 potpie

javac Main.java
// 컴파일 성공, 결과는 pancake

// 매번 컴파일전에 기존 *.class 파일을 모두 삭제
```



  

문제의 해결책은 톱레벨 클래스들을 서로 다른 소스 파일로 분리하는 것으로 아주 간단하다.

굳이 여러 톱레벨 클래스를 한파일에 담고 싶다면 정적 멤버 클래스([아이템 24](Item24.md))를 사용하는 방법을 고민해볼 수 있다.

읽기 좋고, private 으로 선언하면([아이템 15](Item15.md)) 접근 범위도 최소로 관리할 수 있기 때문이다.

  

**정적 멤버 클래스로 바꿔본 예**

```java
public class Test {
  public static void main(String[] args) {
    System.out.println(Utensil.NAME + Dessert.NAME);
  }
  
  private static class Utensil {
    static final String NAME = "pan";
  }
  
  private static class Dessert {
    static final String NAME = "cake";
  }
}
```

  

  

***소스 파일 하나에는 반드시 톱레벨 클래스(혹은 톱레벨 인터페이스)를 하나만 담자.***

이 규칙만 따른다면 컴파일러가 한 클래스에 대한 정의를 여러 개 만들어내는 일은 사라지며, 어떤 순서로 컴파일하든 바이너리 파일이나 프로그램의 동작이 달라지는 일은 결코 일어나지 않을 것이다.

