## 33. 타입 안전 이종 컨테이너를 고려하라

<br>

흔히 사용하는 컬렉션에서의 제네릭을 보면, 매개변수화 되는 대상은 원소가 아닌 컨테이너 자신이다.

하나의 컨테이너에서 매개변수화 할 수 있는 타입의 수가 제한된다.

예를 들어 `Set` 에는 원소 타입을 뜻하는 하나의 매개변수만 있으면 되며, `Map` 에는 키와 값 타입을 뜻하는 2개의 매개변수 타입만 필요하다.

<br>

흔히 사용하는 일반적인 컬렉션은 ***제한된 타입만을 갖게된다.***

```java
Set<String> s = ...;     			 // String 타입만을 담는다 (1가지 타입만 갖게 된다)
Map<Integer, String> m = ...;  // 키와 값 타입의 2가지 타입만을 갖게 된다
```

<br>

### 타입 안전 이종 컨테이너란? 

더 유연한 수단으로, 하나의 컨테이너에서 ***여러 타입의 데이터를 저장 할 수 있는 것*** 을 말한다.

키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하면 된다.

```java
public class HeterogeneousContainer {
	  // 비한정적 와일드 카드 타입Class<?>
    private final Map<Class<?>, Object> items = new HashMap<>(); 

    public <T> void put(Class<T> type, T instance) {
      items.put(Objects.requireNonNull(type), instance);
    }
    public <T> T get(Class<T> type) {
      // cast는 형변환 연산자의 동적버전
      // cast의 반환타입은 Class 객체의 타입 매개변수와 같다
      return type.cast(items.get(type));
    }
}
```

```java
public static void main(String[] args) {
    HeterogeneousContainer container = new HeterogeneousContainer();
    container.put(String.class, "스트링");
    container.put(Integer.class, 20);
    container.put(Float.class, 1.06f);

    System.out.println("여러 타입을 안전하게 넣고 뺄 수 있다.");
    System.out.println("String value : " + container.get(String.class));
    System.out.println("Integer value : " + container.get(Integer.class));
    System.out.println("Float value : " + container.get(Float.class));
}

// 결과
여러 타입을 안전하게 넣고 뺄 수 있다.
String value : 스트링
Integer value : 20
Float value : 1.06
```

<br>

#### 타입 토큰(type token)

컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고 받는 class 리터럴

<br>

#### HeterogeneousContainer 클래스의 제약

1. 악의적인 클라이언트가 Class 객체를 raw 타입으로 넘기면([아이템 26](Item26.md)) 클래스의 인스턴스의 타입 안전성이 쉽게 깨진다.

   HeterogeneousContainer 가 타입 불변식을 어기는 일이 없도록 보장하려만 instance 의 타입이 type 으로 명시한 타입과 같은지 확인한다.

   ```java
   // 동적 형변환으로 런타임 타입 안전성 확보
   public <T> void put(Class<T> type, T instance) {
     	items.put(Objects.requireNonNull(type), type.cast(instance)); // cast 로 타입 확인
   }
   ```

2. 실체화 불가 타입([아이템 28](Item28.md)) 에는 사용할 수 없다.

   String이나 String[] 은 저장할 수 있지만, List<String> 은 저장할 수 없다. List<String> 의 Class 객체를 얻을 수 없기 때문이다.

   List<String> 과 List<Integer> 는 같은 List.class 라는 Class 객체를 사용하므로, 하나의 타입으로 인식되게 된다.

   이 제약에 대한 완벽히 만족스러운 우회로는 없다.

<br>

#### 비한정적 타입 토큰

HeterogeneousContainer 가 사용하는 타입 토큰은 비한정적이다.

put 으로 어떤 Class 객체든 받아들인다.

만약 허용하고 싶은 타입을 제한하고 싶다면, 한정적 타입 토큰을 활용하면 가능하다.

한정적 타입토큰은 한정적 타입매개변수([아이템 29](Item29.md))나 한정적 와일드카드([아이템 31](Item31.md))를 사용하여 표현 가능한 타입을 제한하는 타입 토큰이다.

<br>





