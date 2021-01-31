## Item 37. ordinal 인덱싱 대신 EnumMap을 사용하라

<br>

가끔 배열이나 리스트에서 원소를 꺼낼 때 `ordinal` 메서드([아이템 35](Item35.md))로 인덱스를 얻는 코드가 있다.

이런 코드가 어떤 문제를 일으킬 수 있는지 다음 클래스와 예제 코드를 통해 살펴보자.

<br>

***REMIND: 앞서 [아이템 35](Item35.md)에서도 이야기 했지만, `ordinal` 은 대부분의 프로그래머는 쓸 일이 없어야 한다***

<br>

```java
// 식물 객체
class Plant {
  enum LifeCycle { ANNUAL, PERENNIAL BIENNIAL }
  
  final String name;
  final LifeCycle lifeCycle;
  
  Plant(String name, LifeCycle lifeCycle) {
    this.name = name;
    this.lifeCycle = lifeCycle;
  }
  
  @Override
  public String toString() {
    return name;
  }
}
```

<br>

#### `ordinal()` 을 배열 인덱스로 사용 - 따라하지 말 것!

```java
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for (int i = 0; i < plantsByLifeCycle.length; i++)
  plantsByLifeCycle[i] = new HashSet<>();

for (Plant p : garden)
  plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);  // ordinal 메서드 사용!!

// 결과 출력
for (int i = 0; i < plantsByLifeCycle.length; i++) {
  System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

<br>

#### `ordinal` 을 배열 인덱스로 사용하는 경우 문제점

1. 배열은 제네릭과 호환되지 않으니([아이템 28](Item28.md)) 비검사 형변환을 수행해야 하고 깔끔히 컴파일되지 않을 것이다

2. 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다.

3. 정확한 정수값을 사용한다는 것을 개발자가 직접 보증해야 한다. ***가장 심각한 문제***

   정수는 열거 타입과 달리 타입 안전하지 않기 때문에, 잘못된 값을 사용하면 잘못된 동작을 수행하거나 `ArrayIndexOutOfBoundsException` 을 던질 것이다.

<br>

#### EnumMap을 사용해 데이터와 열거 타입을 매핑하여 개선한 코드

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
for (Plant.LifeCycle lc : Plant.LifeCycle.values())
  plantsByLifeCycle.put(lc, new HashSet<>());
for (Plant p : garden)
  plantsByLifeCycle.get(p.lifeCycle).add(p);
System.out.println(plantsByLifeCycle);
```

**개선하여 얻은 것**

1. 더 간략하고 명료하여 보기 좋다.
2. 타입 안전하다.
3. 맵의 키인 열거 타입이 자체로 출력용 문자열을 제공하여 별도의 레이블을 달지 않아도 된다.
4. 배열 인덱스를 계산하는 과정에서 오류가 날 일이 없다.
5. 성능도 원래 버전(배열)과 비등하다.

<br>

#### EnumMap?

열거 타입을 키로 사용하도록 설계된 Map 구현체

<br>

#### EnumMap 의 구현방식과 성능

EnumMap 의 성능이 `ordinal` 을 쓴 배열에 비견되는 이유는 그 내부에서 배열을 사용하기 때문이다.

내부 구현방식을 안으로 숨겨서 Map 의 타입 안전성과 배열의 성능을 모두 얻어낸 것이다.

<br>

<br>

#### EnumMap 사용의 간단한 예제

```java
public enum DayOfWeek {
  MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

// enum을 키로 사용하는 EnumMap을 생성
Map<DayOfWeek, String> activityMap = new EnumMap<>(DayOfWeek.class);

// add elements
activityMap.put(DayOfWeek.MONDAY, "Running");
activityMap.put(DayOfWeek.WEDNESDAY, "Swimming");
activityMap.put(DayOfWeek.SATURDAY, "Cycling");

// check elements by key
System.out.println(activityMap.containsKey(DayOfWeek.MONDAY)); // true

// check elements by value
System.out.println(activityMap.containsValue("Cycling")); // true
```

[출처](https://www.baeldung.com/java-enum-map)

<br>

***배열의 인덱스를 얻기 위해 `ordinal` 을 쓰는 대신 EnumMap 을 사용하라.***

<br>