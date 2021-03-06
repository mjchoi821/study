## Item 28. 배열보다는 리스트를 사용하라

#### 배열과 제네릭 타입의 두 가지 차이

1. **배열은 공변(covariant)** 이다.

   Sub 가 Super 의 하위타입이라면 배열 Sub[] 는 배열 Super[] 의 하위타입이 된다. (공변, 즉 함께 변환한다는 뜻이다.)

   반면, **제네릭은 불공변(invariant)** 이다. 즉, 서로 다른 타입 Type1 과 Type2 가 있을 때, List<Type1> 은 List<Type2> 의 하위 타입도 상위 타입도 아니다.

   이것만 보면 제네릭에 문제가 있다고 생각할 수도 있지만, 사실 문제가 있는 건 배열 쪽이다. 다음은 문법상 허용되는 코드다.

   ```java
   // 런타임에 실패하는 코드
   Object[] objectArray = new Long[1];
   objectArray[0] = "타입이 달라 넣을 수 없다.";  // ArrayStoreException을 던진다
   
   // 컴파일되지 않는 코드
   List<Object> ol = new ArrayList<Long>(); // 호환되지 않는 타입이다
   ol.add("타입이 달라 넣을 수 없다.");
   ```

   배열에서는 실수를 런타임에야 알게 되지만, 리스트를 사용하면 컴파일타임에 바로 알 수 있다.

2. **배열은 실체화(reify) 된다.**

   배열은 런타임에도 자신이 담는 원소의 타입을 인지하고 확인한다. 그래서 위 예제 코드에서 ArrayStoreException 이 발생한다.

   제네릭은 타입의 정보가 런타임에는 소거(erasure)된다. 원소 타입을 컴파일타임에만 검사하며 런타임에는 알수조차 없다는 뜻이다.

   소거는 제네릭이 지원되기 전의 레거시 코드와 제네릭 타입의 호환성을 위해 사용되는 메커니즘이다.([아이템 26](Item26.md))

<br>

이상의 주요 차이로 인해 배열과 제네릭은 잘 어우러지지 못한다. 

예컨대 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다. 제네릭 배열을 만들지 못하게 막은 이유는 타입 안전하지 않기 때문이다.

이를 허용한다면 컴파일러가 자동 생성한 형변환 코드에서 런타임에 ClassCastException 이 발생할 수 있다. 이는 런타임에 ClassCastException이 발생하는 일을 막아주겠다는 제네릭 타입 시스템의 취지에 어긋나는 것이다.

<br>

#### 실체화 불가 타입(non-reifiable type)

E, List<E>, List<String> 같은 타입을 말한다.

실체화되지 않아서 런타임에는 컴파일타임보다 타입 정보를 적게 가지는 타입이다. 

소거 매커니즘 때문에 매개변수화 타입 가운데 실체화될 수 있는 타입은 List<?> 와 Map<?, ?> 같은 비한정적 와일드카드 타입뿐이다([아이템 26](Item26.md)).

배열을 비한정적 와일드카드 타입으로 만들 수는 있지만, 유용하게 쓰일 일은 거의 없다.

<br>

#### 배열 ⇢ 제네릭

배열을 제네릭으로 만들 수 없어 귀찮을 때도 있다. 제네릭 컬렉션에서는 자신의 원소 타입을 담은 배열을 반환하는게 보통은 불가능하다(완벽하지는 않지만 대부분의 상황에서 이 문제를 해결해주는 방법을 [아이템 33](Item33.md)에서 설명한다).

또 제네릭 타입과 가변인수 메서드(varargs method, [아이템 53](Item53.md)) 를 함께 쓰면 해석하기 어려운 경고 메시지를 받게 된다. 가변인수 메서드를 호출할 때마다 가변인수 매개변수를 담을 배열이 하나 만들어지는데, 이 때 그 배열의 원소가 실체화 불가 타입이라면 경고가 발생하게 된다. 이 문제는 @SafeVarargs 애노테이션으로 대처 할 수 있다([아이템 32](Item32.md)).

<br>

#### 타입 안전성과 상호운용성

배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분은 배열인 E[] 대신 컬렉션인 List<E> 를 사용하면 해결된다. 

코드가 조금 복잡해지고 성능이 살짝 나빠질 수도 있지만, 타입 안전성과 상호운용성은 좋아진다.

<br>

<br>

***배열과 제네릭에는 매우 다른 타입 규칙이 적용된다.***

***따라서 둘을 섞어 쓰기란 쉽지 않다. 둘을 섞어 쓰다가 컴파일 오류나 경고를 만나면, 가장 먼저 배열을 리스트로 대체하는 방법을 적용해보자.***





