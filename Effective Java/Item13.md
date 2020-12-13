## Item 13. clone 재정의는 주의해서 진행하라



Cloneable 은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스(mixin interface, 아이템 20)지만, 아쉽게도 의도한 목적을 제대로 이루지 못했다. 가장 큰 문제는 clone 메서드가 선언된 곳이 Cloneable이 아닌 Objact 이고, 그마저도 protected 라는 데 있다. 그래서 Cloneable 을 구현하는 것만으로 외부 객체에서 clone 메서드를 호출할 수 없다.



### clone 메서드를 잘 동작하게 해주는 구현 방법

#### Cloneable 인터페이스

Cloneable 인터페이스는 Object 의 protected 메서드인 clone 의 동작 방식을 결정한다. Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException 을 던진다.

인터페이스를 구현한다는 것은 일반적으로 해당 클래스가 그 인터페이스에서 정의한 기능을 제공한다고 선언하는 행위다. 그런데 Cloneable의 경우에는 상위 클래스에 정의된 protected 메서드의 동작 방식을 변경한 것이다. 이는 매우 이례적으로 사용한 예이니 따라하지 말자.

Cloneable을 구현한 클래스는 clone 메서드를 public 으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다. 



#### clone 메서드 일반 규약

> 이 객체의 복사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다.
> 일반적인 의도는 다음과 같다. 어떤 객체 x에 대해 다음 식은 참이다.
>
> x.clone() != x
>
> 또한 다음 식도 참이다.
>
> x.clone().getClass() == x.getClass()
>
> 하지만 이상의 요구를 반드시 만족해야 하는 것은 아니다.
> 한편 다음 식도 일반적으로 참이지만, 역시 필수는 아니다.
>
> x.clone().equals(x)
>
> 관례상, 이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야 한다. 이 클래스와 (Object를 제외한) 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.
>
> x.clone().getClass() == x.getClass()
>
> 관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone 으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

강제성이 없다는 점만 빼면 생성자 연쇄(constructor chaining)와 비슷한 매커니즘이다. 즉, clone 메서드가 super.clone이 아닌, 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일러는 불평하지 않을 것이다. 하지만 이 클래스의 하위 클래스에서 super.clone을 호출한다면 잘못된 클래스의 객체가 만들어져, 결국 하위 클래스의 clone 메서드가 제대로 동작하지 않게 된다.(하위 클래스 타입의 객체 생성이 되어야 하는데, clone이 처음 호출된 상위 클래스의 객체가 만들어지게 된다)



#### 불변 클래스는 clone 메서드를 제공하지 않는 게 좋다

 쓸데없는 복사를 지양하는 관점에서 불변 클래스는 clone 메서드는 제공하지 않는 게 좋다. 
PhoneNumber 라는 클래스가 불변 클래스(가변 상태를 참조하지 않는) 라는 가정하에 clone 메서드는 다음처럼 구현할 수 있다.

```java
@Override
public PhoneNumber clone() {
  try {
    return (PhoneNumber) super.clone(); // 자바의 공변 반환 타이핑 덕에 가능하다, 그리고 이 코드는 절대 예외가 일어나지 않는다
  } catch (CloneNotSupportedException e) {
    throw new AssertionError(); // 일어날 수 없는 일, 하지만 Object의 clone 메서드가 이를 강제하고 있다. 이부분이 사실 비검사 예외(unchecked exception)이었어야 했다는 반증이다.
  }
}
```



#### 가변 객체를 복제하는 방법

위의 간단한 구현이 클래스가 가변 객체를 참조하는 순간 재앙으로 돌변한다.
아이템 7에서 소개한 Stack 클래스에 clone 메서드를 추가한다고 생각해보면, 그리고 clone에 의해 복제한다면, elements 필드의 참조는 어떻게 될까?
원본과 같은 배열을 참조하게되어 두 인스턴스가 같은 객체를 수정하게 될 것이다. 따라서 프로그램이 이상하게 동작하거나 NullPointerException을 던지게 될 것이다. 

Stack 의 clone 메서드가 제대로 동작하려면 스택 내부 정보를 복사해야 하는데, 가장 쉬운 방법은 elements 배열의 clone을 재귀적으로 호출해주는 것이다.

```java
@Override
public Stack clone() {
  try {
    Stack result = (Stack) super.clone();
    result.elements = elements.clone(); // 배열의 clone은 런타임 타입과 컴파일타임 타입 모두가 원본 배열과 똑같은 배열을 반환한다
    return result;
  } catch (CloneNotSupportedException e) {
    throw new AssertionError();
  }
}
```

위의 방식은 elements 필드가 final 이었다면 작동하지 않는다. final 필드에는 새로운 값을 할당할 수 없기 때문이다. 이는 근본적인 문제로, 직렬화와 마찬가지로 **Cloneable 아키텍처는 '가변 객체를 참조하는 필드는 final로 선언하라'는 일반 용법과 충돌한다**(단, 원본과 복제된 객체가 가변 객체를 공유해도 안전하다면 괜찮다). 그래서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있다.



clone을 재귀적으로 호출하는 것만으로는 충분하지 않을 때도 있다. 해시테이블용 clone 메서드일 경우가 그렇다.
복잡한 가변 상태를 갖는 클래스의 경우, 특히 해시테이블의 경우 내부 버킷 배열의 참조 이슈(예기치 않게 동작할 가능성)를 해결하기 위해 깊은복사(deep copy)를 이용해 구현하자. 이때 deepCopy 메서드는 연결 리스트 전체를 복사하기 위해 자신을 재귀적으로 호출해도 되지만, 리스트가 길면 스택 오버플로를 일으킬 위험이 있는데, 이 문제를 피하려면 재귀 대신 반복자를 써서 순회하면 된다.





