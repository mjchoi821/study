## Item 29. 이왕이면 제네릭 타입으로 만들라

[아이템 7](Item07.md)에서 다룬 단순한 스택 코드를 제네릭 타입으로 만들어보자.

```java
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }
  
  public void push(Object e) {
    ensureCapacity();
    elements[size++] = e;
  }
  
  public Object pop() {
    if (size == 0)
      throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
  }
  
  public boolean isEmpty() {
    return size == 0;
  }
  
  private void ensureCapacity() {
    if (elements.length == size)
      elements = Arrays.copyOf(elements, 2 * size + 1);
  }
}
```

<br>

#### 변환 방법과 오류 해결

1. 클래스 선언에 타입 매개변수 추가

2. 코드에 쓰인 Object 를 적절한 타입 매개변수로 수정

   - 실체화 불가 타입으로 배열을 만들 수 없어 오류 발생 (제네릭 배열 생성 오류)

     - 해결방법 1 : Object 배열을 생성한 후 제네릭 배열로 형변환(제네릭 배열 생성을 금지하는 제약을 우회하는 방법)

       이 방법은 일반적으로 타입 안전하지 않지만, 비검사 형변환이 안전한지를 직접 확인 후 @suppressWarnings 애노테이션으로 해당 경고를 숨긴다([아이템 27](Item27.md)).

     - 해결방법 2 : elements 필드 타입을 `E[]` 에서 `Object[]` 로 수정

       위와 마찬가지로 비검사 형변환을 수행하는 할당문의 타입 안전성을 확인한 후 경고를 숨길 수 있다.

     > 현업에서는 첫번째 방식을 더 선호하며 자주 사용한다. 하지만 배열의 런타임 타입이 컴파일타임과 달라 힙 오염(heap pollution; [아이템 32](Item32.md)) 을 일으킨다. 힙 오염이 마음에 걸리는 프로그래머는 두번째 방식을 고수하기도 한다.

<br>

<br>

Stack 예처럼 대다수의 제네릭 타입은 타입 매개변수에 아무런 제약을 두지 않는다. 어떤 참조 타입으로도 Stack 을 만들 수 있다. 

단, 기본 타입은 사용할 수 없다. Stack<int> 나 Stack<double> 을 만들려고 하면 컴파일 오류가 난다.

이는 박싱된 기본 타입([아이템 61](Item61.md)) 을 사용해 우회할 수 있다.

<br>

**한정적 타입 매개변수(bounded type parameter)** 는 타입 매개변수에 제약을 두는 제네릭 타입을 말한다.

타입 매개변수 목록의 `<E extends Delayed>` 의 의미는 Delayed 의 하위 타입만 받는다는 뜻이다.

모든 타입은 자기 자신의 하위 타입이므로 `<E extends Delayed>` 의 경우 `SomeClass<Delayed>` 로도 사용할 수 있다.

<br>

<br>

***클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다.***

***따라서 기존 타입 중 제네릭이었어야 하는게 있다면 제네릭 타입으로 변경하자.***