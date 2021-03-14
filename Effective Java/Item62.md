# Item 62. 다른 타입이 적절하다면 문자열 사용을 피하라

<br>

> 문자열(String)은 텍스트를 표현하도록 설계되었고, 그 일을 아주 멋지게 해낸다.
> 그런데 문자열은 워낙 흔하고 자바가 또 잘 지원해주어 원래 의도하지 않은 용도로도 쓰이는 경향이 있다. 

<br>

무분별한 문자열 사용을 주의해야 한다는 경고와 사례들을 이야기해 주고 있다.

### 1. 파일, 네트워크, 키보드 입력으로부터 데이터를 받을 때 다시한번 생각하자

많은 사람이 위의 경우일때 주로 문자열을 사용한다.

사뭇 자연스러워 보이지만, 입력받을 데이터가 진짜 문자열일 때만 그렇게 하는게 좋다.

받은 데이터의 타입을 고민하고 적절한 타입이 있다면 그것을 사용하고, 없다면 새로운 타입을 작성하라.

<br>

### 2. 문자열은 열거 타입을 대신하기에 적합하지 않다

[아이템 34](Item34.md) 에서 이야기했듯, 상수를 열거할 때는 문자열보다 열거 타입이 월등히 낫다.

<br>

### 3. 문자열은 혼합 타입을 대신하기에 적합하지 않다

```java
// 혼합 타입을 문자열로 처리한 부적절한 예
String compoundKey = className + "#" + i.next();
```

위 방식의 단점들은 아래와 같다.

- 두 요소를 구분해주는 문자 `#` 이 두 요소 중 하나에서 쓰였다면 혼란스러운 결과를 초래한다
- 각 요소를 개별로 접근하려면 문자열을 파싱해야 해서 느리고, 귀찮고, 오류 가능성도 커진다
- 적절한 `equals`, `toString`, `compareTo` 메서드를 제공할 수 없으며, String 이 제공하는 기능에만 의존해야 한다
  차라리 전용 클래스를 새로 만드는 편이 낫다 ⇢ private 정적 멤서 클래스로 선언하자([아이템 24](Item24.md))

<br>

### 4. 문자열은 권한을 표현하기에 적합하지 않다

예제는 **스레드의 지역변수 기능** 을 직접 구현한 것인데 자바 2 이전에는 실제로 이렇게 직접 구현하여 사용했다고 한다. 

하지만 이는 문제가 있는 방식이었다.

```java
// 잘못된 예 - 문자열을 사용해 권한을 구분하였다
public class ThreadLocal {
  private ThreadLocal() { } // 객체 생성 불가
  
  // 현 스레드의 값을 키로 구분해 저장한다
  public static void set(String key, Object value);
  
  // (키가 가리키는) 현 스레드의 값을 반환한다
  public static Object get(String key);
}
```

**문제 1.** 스레드 구분용 문자열 키가 전역 이름공간에서 공유된다.

이 방식이 의도대로 동작하려면 각 클라이언트가 고유한 키를 제공해야한다.

그런데 만약 두 클라이언트가 서로 소통하지 못해 같은 키를 쓰기로 결정한다면, 의도치 않게 같은 변수를 공유하게 된다. 

결국 두 클라이언트 모두 제대로 기능하지 못할 것이다.

**문제 2.** 보안에 취약하다.

악의적인 클라이언트라면 의도적으로 같은 키를 사용하여 다른 클라이언트의 값을 가져올 수도 있다.

<br>

**개선 1.** 고유한 키를 사용하도록 개선

문자열 대신 위조할 수 없는 키를 사용하면 해결된다. 이 키를 권한(capacity)이라고도 한다.

```java
public class ThreadLocal {
  private ThreadLocal() { } // 객체 생성 불가
  
  public static class Key { // 권한
    Key() { }
  }
  
  // 위조 불가능한 고유 키를 생성
  public static Key getKey() {
    return new Key();
  }
  
  public static void set(Key key, Object value);
  public static Object get(Key key);
}
```

**개선 2.** set과 get의 개선

`set` 과 `get` 은 이제 정적 메서드일 이유가 없으니 Key 클래스의 인스턴스 메서드로 바꾼다.

이렇게 하면 Key는 더 이상 스레드 지역변수를 구분하기 위한 키가 아니라, 그 자체가 스레드 지역변수가 된다.

결과적으로 `ThreadLocal` 은 하는 일이 없어지므로 , 중첩 클래스 Key 의 이름을 `ThreadLocal` 로 바꾸자. 

최종적으론 아래와 같아진다.

```java
public final class ThreadLocal {
  public ThreadLocal();
  public void set(Object value);
  public Object get();
}
```

**개선 3.** 매개변수화하여 타입안전성 개선

위의 개선 코드에선 `get` 으로 얻은 Object를 실제 타입으로 형변환해서 써야하므로 타입에 안전하지 않다.

따라서 매개변수화 타입([아이템29](Item29.md)) 으로 선언해 타입안전성을 확보하자.

```java
public final class ThreadLocal<T> {
  public ThreadLocal();
  public void set(T value);
  public T get();
}
```

<br>

이렇게 개선한 최종 코드는 `java.lang.ThreadLocal` 과 흡사해졌다.  

문자열 기반 API의 문제를 해결해주며, 키 기반 API보다 빠르고 우아하다.



## 참고

- [ThreadLocal 참고 링크](https://lion-king.tistory.com/entry/Java-ThreadLocal-what-is)

