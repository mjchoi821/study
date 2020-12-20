## Item 12. toString을 항상 재정의하라



### 왜 적용해야 하는가?

Object의 기본 toString 메서드는 단순히 `클래스_이름@16진수로_표시한_해시코드` 를 반환할 뿐이다. 
따라서 우리가 작성한 클래스에 맞게 toString 을 재정의 해야 할 필요가 있다.



#### toString 일반 규약

- 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 한다
- 모든 하위 클래스에서 재정의하라



앞서 나온 equals 와 hashCode (아이템 10, 11)만큼 중요하진 않지만, toString 을 잘 구현한 클래스는 사용하기 훨씬 즐겁고, 그 클래스를 사용한 시스템을 디버깅하기 쉽게 해준다.

toString 메서드는 프로그래머가 직접 호출하지 않더라도 println, printf, 문자열 연결 연산자(+), assert 구문에 넘길 때, 디버거가 객체를 출력할 때 등 자동으로 호출되는 경우가 있다. 따라서 이를 잘 재정의하지 않는다면 프로그래머가 작성한 객체를 참조하는 컴포넌트의 오류 메세지를 쉽게 확인 할 수 없게된다.



```java
// phoneNumber 의 toString 을 잘 정의한 예
System.out.println(phoneNumber + "에 연결할 수 없습니다.");
// "010-1234-1234에 연결할 수 없습니다." 라고 알맞는 내용이 출력 될 것이다

// toString 을 정의하지 않으면
// "PhoneNumber@4d7e1886에 연결할 수 없습니다." 라고 알아볼 수 없게 나온다
```



### 적용시 주의사항

- 객체가 가진 주요 정보 모두를 반환하는게 좋다	
- 반환값의 포맷을 문서화할지 정해야 한다
  포맷을 명시하면 그 객체는 표준적이고, 명확하고, 사람이 읽을 수 있게 된다
  포맷을 명시하기로 했다면, 명시한 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩토리나 생성자를 함께 제공해주면 좋다
  자바 플랫폼의 많은 값 클래스가 따르는 방식이기도 하며 BigInteger, BigDecimal 등이 해당된다
  포맷의 단점은 한번 명시하면 평생 그 포맷에 얽매이게 된다는 점이다, 반대로 이 단점은 포맷을 명시하지 않는다면 향후 릴리즈에서 포맷을 개선할 수 있는 유연성을 얻게 된다고도 말할 수 있다
- toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API 를 외부에 제공하자
  ex) phoneNumber 는 지역 코드, 프리픽스, 가입자 번호용 접근자를 제공해야 한다
  이를 제공하지 않으면, 이 정보가 필요한 부분에서는 toString의 반환값을 파싱할 수밖에 없다. 그렇게 되면 성능이 나빠지고 불필요한 일을 하게되는 것뿐이다

- 하위 클래스들이 공유해야 할 문자열 표현이 있는 추상 클래스
  ex) 대다수의 컬렉션 구현체는 추상 컬렉션 클래스들의 toString 메서드를 상속해 쓴다



### 예외사항

- 정적 유틸리티 클래스([아이템 4](Item04))는 toString을 제공할 이유가 없다
- 열거 타입(아이템 34)도 자바가 이미 완벽한 toString을 제공하니 따로 재정의 하지 않아도 된다


