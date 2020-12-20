## Item 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라



인스턴스 필드들만 존재하는 클래스를 작성할 때, 그리고 그 클래스가 public 이며, 패키지 바깥에서 접근할 수 있는 클래스라면 **접근자를 제공** 하라



**좋은 사용방법이 아닌 클래스**

예를들어 아래와 같은 간단한 x, y 필드만을 갖는 클래스가 있다고 하자.
아래 클래스는 접근자(get/set)를 제공하지 않고 필드를 모두 public 으로 노출하고 있다.

```java
// 좋은방법이 아닌 클래스임을 유의하자
class Point {
  public double x;
  public double y;
}
```

**⇢ 문제점**

- 캡슐화의 이점을 제공하지 못한다 (외부에 모든 내용이 노출되어있기 때문에)
- 외부에서 필드에 접근 시 부수 작업을 수행할 수 없다 (ex. 유효성 검사 등..)
- 내부 표현방식(x, y)을 마음대로 바꿀 수 없다 (이미 외부에서 사용중이기 때문에, 변경 시 외부의 x, y 를 참조하는 모든부분에 수정의 여파가 미친다)



**개선버전**

이번 아이템에서 이야기하는 접근자를 제공하는 형식으로 바꿔보자.

```java
// 필드를 보호하자!
class Point {
  private double x;
  private double y;
  
  public Point(double x, double y) {
    this.x = x;
    this.y = y;
  }
  
  public double getX() { return x; }
  public double getY() { return y; }
  
  public void setX(double x) { this.x = x; }
  public void setY(double y) { this.y = y; }
}
```

**⇢ 얻을 수 있는 이점**

- 클래스 내부 표현방식을 바꿀 수 있다 
  - 유연성이 좋아진다는 것
  - 외부 API 는 유지한 채 내부 x, y 필드 네임을 변경할 수 있다. 수정의 여파가 외부로 퍼지지 않을 수 있다.
  - 여담으로, 위의 클래스는 단순한 값을 넣기때문에 내부 필드명을 그대로 접근자 메서드에 노출시켰지만, 그렇게 하지 않고, 실체적으로 외부에서 필요한 API 명으로 외부와 메시지를 주고받는 API 를 생성하는 편이 좋다.



여기까지의 이야기는 Point 가 패키지 이상에서 접근이 가능한 **public 클래스라는 전제** 하에서의 이야기이다.

만약, ***package-private(default)클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다해도 전혀 문제가 없다.***

그 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 된다. 그리고 public 이 아닌 package-private 또는 private 중첩 클래스인 경우 필드를 public 으로 노출하는 것이 클래스 선언 면에서나 사용하는 클라이언트 코드 면에서나 접근자 방식보다 훨씬 깔끔하다.

필드의 유효영역을 보통은 필드가 속해있는 클래스 한정으로 생각하는 것이라면, 위의 경우는 필드의 유효영역을 패키지 또는 중첩 클래스의 외부 클래스까지로 확장된 형태라 볼 수 있겠다.



**자바 플랫폼 라이브러리의 예외 사례**

`java.awt.package 패키지의 Point 와 Dimension 클래스`

아이템 67에서 설명하듯, 내부를 노출한 Dimension 클래스의 심각한 성능 문제는 오늘날까지도 해결되지 못했다고 한다.



**public 클래스의 필드가 불변이라면?**

직접 노출할 때의 단점이 조금은 줄어들지만, 여전히 결코 좋은 생각이 아니다. 

API를 변경하지 않고는 표현방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전하다.



<br>

### 요약

> public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다.
>
> package-private(default) 클래스나 private 중첩 클래스에서는 종종(불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.