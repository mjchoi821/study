# Item 67. 최적화는 신중히 하라

<br>

### 최적화 격언

> 그 어떤 핑계보다 효율성이라는 이름 아래 행해진 컴퓨팅 죄악이 더 많다(심지어 효율을 높이지도 못하면서).
>   − 윌리엄 울프

<br>

> (전체의 97% 정도인) 자그마한 효율성은 모두 잊자. 섣부른 최적화가 만악의 근원이다.
>   − 도널드 크누스

<br>

> 최적화를 할 때는 다음 두 규칙을 따르라.
> 첫 번째, 하지 마라.
> 두 번째, (전문가 한정) 아직 하지 말라. 다시 말해, 완전히 명백하고 최적화되지 않은 해법을 찾을 때까지는 하지 마라.
>   − M. A. 잭슨

<br>

<br>

최적화는 좋은 결과보다 해로운 결과로 이어지기 쉽다. 
섣불리 진행했다간 ***빠르지도 않고 제대로 동작하지도 않으면서 수정하기는 어려운 소프트웨어*** 를 탄생 시킬수 도 있다.
**빠른 프로그램보다 좋은 프로그램을 작성하라.**

<br>

### 좋은 프로그램을 만들기 위한 우선순위

`견고한 구조 > 성능`

- 성능때문에 견고한 구조를 희생하지 말 것
- 견고한 구조의 프로그램에서 원하는 성능이 나오지 않는다면 아키텍처가 최적화 할 수 있는 길을 안내 할 것

<br>

### 좋은 프로그램?

- 정보 은닉 원칙을 따른다 
  - 개별 구성요소의 내부를 독립적으로 설계할 수 있으며, 시스템의 나머지에 영향을 주지 않고 각 요소를 다시 설계할 수 있다([아이템 15](Item15.md))

- 설계 단계에서 성능을 고려한다

  API, 네트워크 프로토콜, 영구 저장용 데이터 포맷 등의 설계 요소들을 고려하자

  - API 설계 시  성능에 주는 영향을 고려하자

    public 타입을 가변으로 만들면, 즉 내부 데이터를 변경할 수 있게 만들면 불필요한 방어적 복사를 수없이 유발할 수 있다([아이템 50](Item50.md))

    합성으로 해결할 수 있음에도 상속 방식으로 설계한 public 클래스는 상위 클래스에 영원히 종속되며 성능 제약까지도 물려받게 된다([아이템18](Item18.md))

    인터페이스가 존재하는데 구현 타입을 사용하는 것 역시 좋지 않다. 특정 구현체에 종속되게 하여, 나중에 다른 구현체가 나오더라도 이용하지 못하게 된다([아이템 64](Item64.md))

    ⇢ 예) `java.awt.Component` 클래스의 `getSize` 메서드
    `getSize`  시 `Dimension` 을 리턴하는데 가변이기 때문에 인스턴스를 방어적 복사하여 리턴, `getSize` 를 사용하는 모든곳에서 인스턴스가 새로 생성되어 문제가 될 수 있는 여지가 있음(수백만 개의 `getSize` 호출 시)
    추후 `getWidth` 와 `getHeight` 로 각각을 호출하는 함수로 나누었으나 어딘가의 기존 클라이언트 코드는 여전히 `getSize` 를 호출할 것이다

  - 성능을 위해 API 를 왜곡하지 말자

    잘 설계된 API는 성능도 좋다

    왜곡하게 될 경우 그를 유지하는데 드는 비용이 더 크다

<br>

### 최적화 전후로 성능을 측정하자

- 프로파일링 도구를 이용하여 최적화를 진행할 부분을 찾자

- 자바에서의 최적화 전후 성능 측정의 중요성

  - 프로그래머가 작성하는 코드와 CPU에서 수행하는 명령 사이의 '추상화 격차'가 커서 최적화로 인한 성능 변화를 일정하게 예측하기가 더 어렵다

  - 자바의 성능 모델은 정교하지 않을뿐더러 구현 시스템, 릴리즈, 프로세서마다 차이가 있다

    따라서 여러 자바 플랫폼이나 여러 하드웨어 플랫폼에서 구동한다면 최적화의 효과를 각각에서 모두 측정해야 한다

    ⇢ 다른 구현 혹은 하드웨어 플랫폼 사이에서 성능을 타협해야 하는 상황도 마주할 것이다

<br>

### 결론

- 성능을 위한 최적화는 설계 단계부터 고려하자
- 하지만 성능때문에 견고한 구조를 희생하진 말자
- 최적화는 프로파일링 도구를 이용해 문제의 원인이 되는 지점을 찾아 최적화를 수행하고, 변경 전과 후의 최적화 효과를 측정하자

<br>

<br>

### 참고

- [짧은 코드를 짜라](https://m.blog.naver.com/knix008/222266419265)

  최적화, 테스트에 대한 이야기인데 마지막에 이야기하는 '구조화가 최적화를 돕는다.' 라는 부분이 본 아이템과 일맥상통하는 부분이 있어 참고링크로 넣어두었습니다. :) 

<br>
