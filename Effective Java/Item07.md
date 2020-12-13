## Item 7. 다 쓴 객체 참조를 해제하라



JVM의 가비지 컬렉터는 더 이상 사용되지 않는 객체를 회수해 적절한 시점에 메모리를 확보한다.
그래서 자칫 메모리 관리에 더 이상 신경쓰지 않아도 된다고 오해할 수 있는데, 이는 사실이 아니다.



아래 예제를 보자. 어디가 잘 못 됐을까?

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
    return elements[--size];
  }
  
  /**
  * 원소를 위한 공간을 적어도 하나 이상 확보한다.
  * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
  */
  private void ensureCapacity() {
    if (elements.length == size)
      elements = Arrays.copyOf(elements, 2 * size + 1);
  }
}
```

위 코드는 꼼꼼한 테스트코드를 만들어봐도 모두 통과할 수 있다.
그럼 문제가 없는걸까? 아니다. 위 코드는 시스템을 오래 사용하면 사용할 수록 문제가 생길 수 있는 여지가 생기는 코드이다.
그 문제는 바로 '메모리 누수' 이다.

메모리 누수는 오래 누적되면 성능이 느려지거나 OutOfMemoryError 와 함께 종료될 수도 있다.

문제는 메서드 pop 에서 발생한다. pop은 Stack 의 가장 마지막요소를 전달해주며 내부에서 사용하는 포인터값(size)도 관리하여, 자기 역할을 다하는 것 처럼 보이지만 자신이 가르키는 index 만 줄어들 뿐, 실제로 반환한 요소를 내부에 그대로 담고있다.
때문에 가비지 컬렉터는 마지막 요소도 계속 사용하는 객체로 인지해 회수하지 않는다.
이와 같은 메모리 누수 문제는 찾기가 아주 까다롭다. 
따라서 코드 작성 시 더이상 사용하지 않는 객체에 대한 참조 해제가 언제 이루어질지도 생각해 봐야 한다.



위의 코드를 아래와 같이 구현하면 메모리 누수 문제를 해결할 수 있다.

```java
public Object pop() {
  if (size == 0)
    throw new EmptyStackException();
  Object result = elements[--size];
  elements[size] = null; // 다 쓴 참조 해제
  return result;
}
// 실제로 java Stack 의 pop 메서드는 Vector 의 removeElementAt 메서드를 사용하는데, 해당 코드에서도 마지막에
// elementData[elementCount] = null; /* to let gc do its work */
// 라고 되어있는 걸 볼 수 있다.
```



이렇게 위 예제에선 다 쓴 객체를 null 처리해주므로써 메모리 누수를 해결했다.
객체 참조를 null 처리 하는 일은 예외적인 경우여야 한다. 모든 객체를 일일이 null 처리하게 되면 코드가 필요 이상으로 지저분해 질 것이다.
다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효범위(scope) 밖으로 밀어내는 것이다.
이 일은 변수의 범위를 최소가 되게 정의했다면(아이템 57) 자연스럽게 이뤄진다.

위의 Stack 에서 메모리 누수가 생긴 이유는 스택이 자기 메모리를 직접 관리하기 때문이다.
프로그래머만이 size 보다 큰곳에 위치한 객체는 유효하지 않다는 걸 알 수 있기에 size 보다 큰곳에 위치한 객체를 더이상 쓰지 않을 거라고 가비지 컬렉터에 알려야 하는 것이다.

그 외에도 메모리 누수가 생길 수 있는 방식은 캐시, 리스너(혹은 콜백)이 있는데 WeakHashMap 을 이용해 약한 참조로 저장해 사용하면 문제를 방지할 수 있다.