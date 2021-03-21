# Item 70. 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라

<br>

![Exception hierarchy](https://lh5.googleusercontent.com/WqqNoyFEkZXfmZBBQjgIutY72_BUV6_By_BAe7Ih9u36HfelS3nTWQEYtdRUkQS32Tuhg9P9CUXo-jgvOpkO84vLm2viI4Od0BNustwONdMm7DKZnKC6kyVHyRJbsESLIPV4uBU)

출처 : https://www.javamadesoeasy.com/2015/05/exception-handling-exception-hierarchy.html

<br>

### 예외 선택, 사용 참고 지침

1. 호출하는 쪽에서 복구하리라 여겨지는 상황이라면 검사 예외를 사용하라.

   - 검사와 비검사 예외를 구분하는 기본 규칙

   - 검사 예외를 던지면 호출자가 그 예외를 `catch` 로 잡아 처리하거나 더 바깥으로 전파하도록 강제하게 됨

   ⇢ 따라서 메서드 선언에 포함된 검사 예외는 그 메서드를 호출했을 때 발생할 수 있는 유력한 결과임을 API 사용자에게 알려주는 것

   또는 API 설계자가 API 사용자에게 검사 예외를 던져주어 해당 상황에서 recovery 를 요구한 것

2. 오류를 나타낼 때는 런타임 예외를 사용하자.

   - 런타임 예외의 대부분은 전제조건을 만족하지 못했을 때 발생

   - 복구할 수 있는 상황인지 프로그래밍 오류인지 명확히 구분되지 않음

     ex) 자원 고갈은 말도 안 되는 크기의 배열을 할당해 생긴 프로그래밍 오류일 수도 있고 진짜로 자원이 부족해서 발생한 문제일 수 있다.

     이 경우, 해당 상황이 복구될 수 있는 것인지는 API 설계자의 판단에 달렸다. 복구가 가능하다고 생각하면 검사 예외를 그렇지 않다면 런타임 예외를 사용하자. 확신하기 어렵다면 비검사 예외를 선택하는 편이 나을 것이다. (이유는 [아이템 71](Item71.md) 참고)

3. 에러를 상속해 하위 클래스를 구현하지 말자.

   - 에러는 보통 JVM 의 자원 부족, 불변식 깨짐 등 더 이상 수행을 계속할 수 없는 상황을 나타낼 때 사용 (업계에 널리 퍼진 규약)

   ⇢ 따라서 Error 클래스를 상속해 하위 클래스를 만들지 말아야 한다

   일반 프로그래머가 구현하는 비검사 `throwable` 은 모두 `RuntimeException` 의 하위 클래스여야 한다

   추가로 에러는 `throw` 문으로 직접 던지는 일도 없어야 한다 (`AssertionError` 는 예외)

4. `Exception`, `RuntimeException`, `Error` 를 상속하지 않는 `throwable` 을 만드는 것도 가능하지만, 이로울게 없으니 **절대 사용하지 말자!**

   - `throwable`  은 정상적인 검사 예외보다 나을게 하나도 없으면서 API 사용자들을 헷갈리게 할 뿐이다

<br>

<br>

### 핵심 정리

**checked exeption (검사 예외)**

- 복구할 수 있는 상황일 때 사용, 복구에 필요한 정보를 알려주는 메서드도 제공할 것 ([아이템 75](Item75.md) 참고)

**unchecked exception (비검사 예외)**

- 프로그래밍 오류
- 확실하지 않을 경우

<br>