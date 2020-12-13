## Item 6. 불필요한 객체 생성을 피하라



모든 객체를 생성하려 하지 말자.
재사용은 빠르고 세련되다. 특히 불변 객체(아이템 17)는 언제든 재사용할 수 있다.
재사용 가능한 것은 재사용하자. 아래 여러 예를들어 설명하고 있다.



### 문자열 리터럴

```java
// 하지 말아야 할 객체 생성의 예
String bad_string = new String("book"); // 해당 코드가 반복 실행될 경우 매번 book 인스턴스를 생성한다

// 아래와 같이 사용해야 한다
String good_string = "book"; // 반복 사용되어도 기존 문자열 리터럴을 재사용한다
```



### 불변 클래스에서 정적 팩토리 메서드 제공 시 

```java
// 하지 말아야 할 방식, 이건 심지어 deprecated 되어있다
Boolean bad_boolean = new Boolean("true"); // new 연산자는 매번 새로운 인스턴스를 생성한다

// 아래와 같이 사용해야 한다
Boolean good_boolean = Boolean.valueOf("true"); // public static final 인스턴스를 리턴한다
```



### 생성 비용이 비싼 객체라면 생성에 대해 한번 더 고민하라 (feat. 정규표현식 Pattern 인스턴스)

- Before

```java
// 보통 유틸 클래스에 들어있을법한 메서드, 아래 유틸메서드는 호출 될 때마다 매번 정규표현식 Pattern 인스턴스를 생성한다
// 정규표현식 Pattern 인스턴스는 정규표현식에 해당하는 유한 상태 머신(finite state machine)을 만들기 때문에 인스턴스 생성 비용이 높다
static boolean isRomanNumeral(String s) {
  return s.matches("^(?=.)M*(C[MD]|D?C{0,3})" 
                   + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

따라서 위의 코드는 아래와 같이 개선하여 성능을 개선 할 수 있다.

- After

```java
public class RomanNumerals {
  private static final Pattern ROMAN = Pattern.compile(
    "^(?=.)M*(C[MD]|D?C{0,3})" 
    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$"
  );
  
  static boolean isRomanNumeral(String s) {
    return ROMAN.matcher(s).matches(); // Pattern 인스턴스를 한번만 생성한 후 매번 재사용한다
  }
}
```

추가적으로 위의 개선된 코드에서는 Pattern 의 이름을 지어주므로써 코드의 의미가 더 잘 들어난다는 추가적인 이점도 얻었다.
하지만 개선된 방식에서 RomanNumerals 클래스가 초기화 된 후 isRomanNumeral 메서드가 한번도 쓰이지 않는다면, ROMAN 필드는 쓸데 없이 초기화되었다고도 볼 수 있는데, 이런부분을 메서드가 처음 호출될 때 필드를 초기화하는 지연 초기화(lazy initialization, 아이템 83)로 없앨 수 있지만 권하지 않는다.
지연 초기화는 코드를 복잡하게 만들지만, 성능은 크게 개선되지 않을 때가 많기 때문이다(아이템 67).
개발은 항상 트레이드 오프임을 잊지 말자.



### 오토박싱(auto boxing)

오토박싱은 기본 타입과 박싱된 기본 타입을 섞어 쓸때 자동으로 상호 변환해주는 기술이다.

```java
private static long sum() {
  Long sum = 0L; // 아마도 객체 Long 으로 타입을 지정한것은 오타일 것이다.. primitive long 타입으로 써야 한다
  for (long i=0; i<=Integer.MAX_VALUE, i++)
    sum += i; // sum 이 Long 타입이기 때문에 i가 더해질 때마다 i는 Long 인스턴스로 변환되어 계산되기에 몇배는 느리게 계산된다
  
  return sum;
}
```

위의 예는 의도치 않은 오토박싱이 숨어들지 않게 주의하자는 이야기 이다.



<br>

이 정도의 예를 들며 책은 이번 아이템을 설명하고 있다.
프로그램의 명확성, 간결성, 기능을 위해서 객체를 추가로 생성하는 것이라면 일반적으로 좋은 일이라고도 이야기한다.
무조건 객체 생성을 피하라는 이야기가 아닌걸 이해했다면, 이번장의 이해는 끝이다.

