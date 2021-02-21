## Item 47. 반환 타입으로는 스트림보다 컬렉션이 낫다

<br>

**자바 7까지 메서드 반환 타입을 결정하는 방법**

스트림이 나오기 전까지 일련의 원소를 반환하는 메서드가 반환 타입을 결정하는 방법은 아래와 같았다.

- 기본적으로 **Collection 인터페이스** 고려(Collection, List, Set)
- 반환된 원소 시퀀스가 일부 Collection 메서드를 구현할 수 없을때 **Iterable 인터페이스** 사용

- 반환 원소들이 기본 타입이거나 성능에 민감한 상황이라면 **배열** 사용

<br>

**자바 8에서 스트림이 생기며 반환 타입을 결정하는일이 복잡해진 이유**

Stream 과 Iterable 의 반복이 서로 호환되지 않기때문에 아래와 같은 문제가 발생한다.

- Stream 만 반환하면 반환된 스트림을 for-each 로 반복하길 원하는 사용자는 불만을 토로하게 됨
- 반대로 Iterable 만 반환하면 스트림 파이프라인으로 처리하려는 사용자가 불만을 가지게 됨

<br>

<br>

그럼 이런 문제를 어떻게 해결해야 할까?

⇢ 멋진 우회로는 없지만, 해결해보자.
<br>

### Stram 을 for-each 로 반복하기

**컴파일 오류 발생코드**  - 형변환하지 않아 컴파일 오류가 발생한다

```java
for (ProcessHandle ph : ProcessHandle.allProcessed()::iterator) {  // 컴파일 오류 발생
  // ...
}
```

<br>

**컴파일 오류가 발생하지 않는 코드**  - 메서드 참조를 매개변수화 된 Iterable 로 적절히 형변환하여 정상동작

```java
for (ProcessHandle ph : (Iterable<ProcessHandle>)ProcessHandle.allProcessed()::iterator) {
  // 하지만 코드가 복잡하다
  // ...
}
```

<br>

**어댑터 메서드를 만들어 개선한 코드**

```java
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
  return stream::iterator; // 자바의 타입 추론이 문맥을 잘 파악하여, 여기서 따로 형변환을 하지 않아도 된다
}

// 깔끔하게 사용할 수 있다
for (ProcessHandle p : iterableOf(ProcessHandle.allProcesses())) {
  // ...
}
```

<br>

### Iterable 을 Stream 으로 변환

```java
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
  return StreamSupport.stream(iterable.spliterator(), false);
}
```

- StreamSupport 클래스는 1.8 에서 추가된 클래스로 다양한 스트림 변환을 지원한다 ([참고](https://docs.oracle.com/javase/8/docs/api/java/util/stream/StreamSupport.html))

<br>

<br>

- 작성하는 메서드가 오직 스트림 파이프라인에서만 쓰일 걸 안다면, 스트림을 반환하자

- 반환된 객체들이 반복문에서만 쓰일 걸 안다면 Iterable 을 반환하자

- 공개 API를 작성할 때는 스트림 파이프라인을 사용하는 사람과 반복문에서 쓰려는 사람 모두를 배려해야 한다

  ⇢ Collection 인터페이스는 Interface 의 하위 타입이고 stream 메서드도 제공하니 반복과 스트림을 동시에 지원한다

  따라서 ***원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는게 일반적으로 최선이다***

  - Arrays 도 `Arrays.asList` 와 `Stream.of` 메서드로 손쉽게 반복과 스트림을 지원한다

  > **주의할 점**
  >
  > 반환하는 시퀀스의 크기가 메모리에 올려도 안전할 만큼 작다면 ArrayList 나 HashSet 같은 표준 컬렉션 구현체를 반환하는게 최선일 수 있다.
  > 하지만 단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안 된다.

<br>

### 전용 컬렉션 구현 예제

반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면, 전용 컬렉션을 구현하는 방안을 검토해보자.

예제는 주어진 집합의 멱집합(한 집합의 모든 부분집합을 원소로 하는 집합)을 반환하는 코드이다.

원소의 개수가 n 개면 멱집합의 원소 개수는 2<sup>n</sup> 개가 된다. 따라서 표준 컬렉션 구현체에 저장하려는 생각은 위험하다.

따라서 `AbstractList` 를 이용하여 전용 컬렉션을 구현해보자.

**ex)** `{a, b, c}` 의 멱집합은 `{{}, {a}, {b}, {c}, {a, b}, {a, c}, {b, c}, {a, b, c}}` 

- 각 원소의 인덱스를 비트 벡터로 사용

  - 인덱스의 n번째 비트 값은 멱집합의 해당 원소가 원래 집합의 n번째 원소를 포함하는지 여부를 알려준다

    ⇢ 0부터 2<sup>n</sup> - 1 까지의 이진수와 원소 n개인 집합의 멱집합과 자연스럽게 매핑된다

```java
public class PowerSet {
  public static final <E> Collection<Set<E>> of(Set<E> s) {
    List<E> src = new ArrayList<>(s);
    if (src.size() > 30)
      throw new IllegalArgumentException("집합에 원소가 너무 많습니다(최대 30개).: " + s);
    // 위의 예외조항은 collection을 반환타입으로 쓸 때의 단점..!(코드 밑에 추가 설명 참조)
    
    return new AbstractList<Set<E>>() {
      @Override public int size() {
        return 1 << src.size();  // 멱집합의 크기는 2를 원래 집합의 원소 수만큼 거듭제곱한 것과 같다
      }
      
      @Override public boolean contains(Object o) {
        return o instanceof Set && src.containsAll((Set)o);
      }
      
      @Override public Set<E> get(int index) {
        Set<E> result = new HashSet<>();
        for (int i = 0; index != 0; i++, index >>= 1)
          if ((index & 1) == 1)
            result.add(src.get(i));
        return result;
      }
    };
  }
}
```

- Collection 을 반환타입으로 쓸 때의 단점

  Collection 의 size 메서드가 int 값을 반환하므로 위 예제의 `PowerSet.of` 가 반환하는 시퀀스의 최대 길이는 `Integer.MAX_VALUE` 혹은 2<sup>31</sup> - 1 로 제한된다.

  Collection 명세에 따르면 컬렉션이 더 크거나 심지어 무한대일 때 size가 2<sup>31</sup> - 1 을 반환해도 되지만 완전히 만족스러운 해법은 아니다.

  ⇢ **Collection 의 반환 size 를 유의해야 한다**

- 반복이 시작되기 전에 시퀀스의 내용을 확정할 수 없는 등의 사유로 `contains` 와 `size` 를 구현하는게 불가능할 때는 컬렉션보다는 스트림이나 Iterable 을 반환하는 편이 낫다(원한다면 별도의 메서드를 두어 두 방식 모두 제공해도 된다)

<br>

### 입력 리스트의 연속적인 모든 부분리스트를 반환하는 예제

#### 스트림 구현 예제 1.

```java
public class SubLists {
  public static <E> Stream<List<E>> of(List<E> list) {
    return Stream.concat(Stream.of(Collections.emptyList()), 
                         prefixes(list).flatMap(SubLists::suffixes));
  }
  
  private static <E> Stream<List<E>> prefixes(List<E> list) {
    return IntStream.rangeClosed(1, list.size())
      .mapToObj(end -> list.subList(0, end));
  }
  
  private static <E> Stream<List<E>> suffixes(List<E> list) {
    return IntStream.range(0, list.size())
      .mapToObj(start -> list.subList(start, list.size()));
  }
}
```

<br>

#### 스트림 구현 예 2.

```java
public static <E> Stream<List<E>> of(List<E> list) {
  return IntStream.range(o, list.size())
    .mapToObj(start -> 
              IntStream.rangeClosed(start + 1, list.size())
              	.mapToObj(end -> list.subList(start, end)))
    .flatMap(x -> x);
}
```

- 빈 리스트를 반환하려면, `concat` 을 사용하거나 `rangeClosed` 호출 코드의 1을 `Math.signum(start)` 로 고쳐주면된다

<br>

스트림을 반환하는 두 가지 구현을 만들어보았다.

하지만 이는 반복을 사용하는게 더 자연스러운 상황에서도 사용자는 스트림을 쓰거나 `Iterable` 로 변환하는 어댑터를 써야 한다. 하지만 어댑터는 클라이언트 코드를 어수선하게 만들고, 저자의 테스트 시 동작이 2.3배가 더 느린 결과가 나왔다.

<br>

<br>

### 정리

원소 시퀀스를 반환하는 메서드를 작성할 때는, 스트림과  반복 두가지 모두 처리될 수 있도록 고려하자.

보통은 아래의 순서를 따르면 좋다.

1. 컬렉션을 반환할 수 있다면 반환한다
2. 반환 전부터 이미 원소들을 컬렉션에 담아 관리하고 있거나 원소 개수가 적다면 `ArrayList` 같은 표준 컬렉션에 담아 반환하자
3. 그렇지 않으면 전용 컬렉션 구현을 고려한다
4. 컬렉션을 반환하는게 불가능하면 스트림과 `Iterable` 중 더 자연스러운 것을 반환하라

<br>

<br>