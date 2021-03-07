# Item 58. 전통적인 for 문보다는 for-each 문을 사용하라

### 전통적인 for문 사용

```java
// Collection 순회
for (Iterator<Element> i = c.iterator(); i.hasNext();) {
  Element e = i.next();
  // ... e 사용
}

// Array 순회
for (int i = 0; i < a.length; i++) {
  // ... a[i] 사용
}
```

- while문 보다는 낫지만([아이템57](Item57.md)) 가장 좋은 방법은 아니다

  - 반복자와 인덱스 변수는 코드를 복잡하게 할 뿐 우리에게 진짜 필요한 건 원소들뿐
  - 쓰이는 요소 종류가 늘어나면 오류가 생길 가능성이 높아진다
    - 잘못된 변수를 사용했을 때, 컴파일러가 오류를 발견하지 못할수 있다
  - 순회하는 대상 컨테이너(Collection, Array)에 따라 코드 형태가 다르다

  ⇢ 위 문제들은 `for-each`문 을 사용하면 모두 해결된다

<br>

### 향상된 for문인 for-each문 사용

- for-each문의 정식 이름은 향상된 for문(enhanced for statement) 이다
- 반복자와 인덱스 변수를 사요하지 않아서 코드가 깔끔해지고 오류가 날 일도 없다(IndexOutOfBoundsException 등..)
- 하나의 관용구로 Collection과 Array를 모두 처리할 수 있어서 어떤 컨테이너를 사용할지 신경쓰지 않아도 된다

```java
for (Element e : element) {  // 콜론(:)은 '안의(in)'라고 읽으면 된다 = 'elements 안의 각 원소 e에 대해'라고 읽는다
  // ... e 사용
}
```

⇢ `for-each`문이 만들어내는 코드는 사람이 손으로 최적화한 것과 사실상 같다

<br>

컬렉션을 중첩해 순회해야 한다면 `for-each`문의 이점이 더욱 커진다.

```java
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING }

// 컬렉션이나 배열의 중첩 반복을 위한 권장 관용구
for (Suit suit : suits) {
  for (Rank rank : ranks) {
    deck.add(new Card(suit, rank));
  }
}
```

<br>

### for-each문을 사용할 수 없는 상황

- 파괴적인 필터링(destructive filtering)

  컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 한다

  자바 8부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다

- 변형(transforming)

  리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다

- 병렬 반복(parallel iteration)

  여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다

위의 세 가지 상황 중 하나에 속할 때는 일반적인 `for`문을 사용하되 이번 아이템에서 언급한 문제들을 경계해야 한다.

<br>

```java
// Iterable 인터페이스
public interface Iterable<E> {
  Iterator<E> iterator();
}
```

- `for-each`문은 Iteraable 인터페이스를 구현한 객체는 모두 순회할 수 있다
- 원소들의 묶음을 표현하는 타입을 작성해야 한다면 `Iterable`을 구현하는 쪽으로 고민해보기 바란다(까다로움 주의)

<br>

### 정리

전통적인 `for`문과 비교했을 때 `for-each`문은 명료하고 유연하고, 버그를 예방해준다.

성능 저하도 없다. 가능한 모든 곳에서 `for`문이 아닌 `for-each`문을 사용하자.