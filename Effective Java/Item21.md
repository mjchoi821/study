## Item 21. 인터페이스는 구현하는 쪽을 생각해 설계하라

  

### 인터페이스에 추가 된 디폴트 메서드

- 자바 8에서 추가
- 기존 인터페이스에 메서드를 넣을 수 있게 됨
- 위 내용이 장점이자 단점, 기존 구현체들과 매끄럽게 연동되지 않을 수 있음

  

### 기존 인터페이스에 디폴트 메서드 사용 시 주의할 점

1. 기존 구현체에 런타임 오류를 일으킬 수 있다
   ⇢ 기존 구현체들과의 충돌하지 않음을 보장할 수 없기 때문에
2. 인터페이스의 메서드를 제거하거나 기존 메서드의 시그니처를 수정하는 용도가 아님을 명심해야 한다
   ⇢ 이런 형태로 인터페이스를 변경하면, 반드시 기존 클라이언트를 망가뜨리게 된다

> 기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니면 피해야 한다

  

#### 추후에 추가된 디폴트 메서드로 인해 예기치 못한 동작을 하는 예

**Apache Commons 의 SynchronizedCollection 클래스** 

> org.apache.commons.collections4.collection.SynchronizedCollection

해당 클래스의 목적은 클라이언트가 제공한 객체를 락 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼 클래스이다.
하지만 자바 8의 Collection 인터페이스에 추가된 디폴트 메서드인 removeIf 메서드를 재정의 하지 않았기 때문에, 자바 8과 함께 사용된다면 디폴트 메서드의 removeIf 를 그대로 사용할 수 있게된다.
그럴경우 본래 클래스의 목적인 제공받은 객체로 락을 처리할 수 없게되어, 예외가 발생하거나 의도하지 않은 동작으로 이어지게 된다.

  

위의 이슈는 4.4 버전부터 재정의 되었다.

[Apache Commons 4.4 API 참고](https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/collection/SynchronizedCollection.html#removeIf-java.util.function.Predicate-)

> #### removeIf
>
> ```
> public boolean removeIf(Predicate<? super E> filter)
> ```
>
> - **Specified by:**
>
>   `removeIf` in interface `Collection<E>`
>
> - **Since:**
>
>   4.4

  

### 새로운 인터페이스 생성 시

- 표준적인 메서드 구현을 제공하는데 유용하며, 인터페이스를 더 쉽게 구현해 활용할 수 있게끔 해준다 ([아이템 20](Item20.md))
- 테스트 시, 최소 세가지 다른 방식으로 구현해봐야 하며, 인스턴스를 활용하는 클라이언트도 여럿 만들어봐야 한다

  

### 개인적인 생각

해당 아이템에서 이야기하는 문제점은 클래스의 상속에서 일어나는 문제와 동일했다. 
상위 클래스의 수정의 여파가 모든 하위 클래스에 미치게되므로 상위 클래스의 수정은 신중해야 하는부분과 동일하며, 구체적인 내용이 들어가는 부분은 확실히 쉽게 변화할 수 없다는걸 다시한번 느끼게 됐다.