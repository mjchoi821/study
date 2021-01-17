## Item 27. 비검사 경고를 제거하라

할 수 있는 한 모든 비검사 경고를 제거하라.

대부분의 비검사 경고는 쉽게 제거할 수 있으며 모두 제거한다면 그 코드는 타입 안전성이 보장된다.

<br>

#### 타입 매개변수 추론

```java
// 잘못 작성된 코드
Set<SomeObject> items = new HashSet();
```

자바 7부터 지원하는 다이아몬드 연산자(<>)로 해결할 수 있다.

```java
Set<SomeObject> items = new HashSet<>();
```

<br>

#### @SuppressWarnings 애노테이션 사용

경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 `@SuppressWarnings("unchecked")` 애노테이션을 달아 경고를 숨기자.

단, 반드시 타입 안전함을 검증해야 한다.

한편 안전하다고 검증된 비검사 경고를 그대로 두면, 진짜 문제를 알리는 새로운 경고가 나와도 눈치채지 못할 수 있다. 따라서 안전함이 확인된 부분은 경고를 제거해주자.

**추가로 유의할 점들**

- 가능한 한 좁은 범위에 적용하자

  @SuppressWarnings 애노테이션은 개별 지역변수 선언부터 클래스 전체까지 어떤 **선언** 에도 달 수 있다.

  자칫 심각한 경고를 놓칠 수 있으니 절대로 클래스 전체에 적용해서는 안 된다.

- 한 줄이 넘는 메서드나 생성자에 달린 @SuppressWarnings 애너테이션을 발견하면 지역변수 선언 쪽으로 옮기자.

  이를 위해 지역변수를 새로 선언하는 수고를 해야 할 수도 있겠지만, 그만한 값어치가 있을 것이다.

  ```java
  public <T> T[] toArray(T[] a) {
    if (a.length < size) 
      return (T[]) Arrays.copyOf(elements, size, a.getClass());   // 컴파일 시 이 곳에서 경고가 발생한다
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size) 
      a[size] = null;
    return a;
  }
  ```

  **지역변수를 추가해 @SuppressWarnings의 범위를 좁힌다.**

  ```java
  public <T> T[] toArray(T[] a) {
    if (a.length < size) {
      // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로
      // 올바른 형변환이다.
      @SuppressWarnings("unchecked") T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
      return result;
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size) 
      a[size] = null;
    return a;
  }
  ```

- 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다

  다른 사람이 그 코드를 이해하는 데 도움이 되며, 더 중요하게는, 다른 사람이 그 코드를 잘못 수정하여 타입 안전성을 잃는 상황을 줄여준다.

  코드가 안전한 근거가 쉽게 떠오르지 않더라도 끝까지 포기하지 말자.

<br>

<br>

***모든 비검사 경고는 런타임에 ClassCastException을 일으킬 수 있는 가능성이 있으니 최선을 다해 제거하라.***

