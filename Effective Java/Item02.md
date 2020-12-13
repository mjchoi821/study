## Item 2. 생성자에 매개변수가 많다면 빌더를 고려하라

> Builder 패턴에 대한 이야기



선택적 매개변수가 4개 이상이고, 타입이 같은 매개변수가 연달아 있을 경우 대개 그 복잡성을 해결하기 쉽지 않다.

이 경우 좋은 해결방안으로 Builder 패턴을 고려해보자. 

Builder 패턴을 사용하면 유연하면서도 가독성이 좋고, 안전한 클래스를 생성 할 수 있다.

단점으로는 생성을 위한 빌더패턴을 따로 구현해야 하기에 코드가 장황해질 수 있다. 따라서 매개변수에 의한 복잡성의 극복을 위해서 또는 앞으로 매개변수가 많이 늘어날 가능성이 있을 경우 등의 상황을 고려하여 선택해야 할 것이다.



- 사용 예 - 빌더 패턴

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값 초기화
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int value) {
            calories = value;
            return this;
        }

        public Builder fat(int value) {
            fat = value;
            return this;
        }

        public Builder sodium(int value) {
            sodium = value;
            return this;
        }

        public Builder carbohydrate(int value) {
            carbohydrate = value;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    public NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```



- 빌더 패턴 사용코드

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100)
                .sodium(35)
                .carbohydrate(27)
                .build();
```



이와 같이 빌더 패턴으로 구현 시 유연하게 데이터를 받아 build 메서드로 인스턴스를 생성하기 때문에, 객체를 안전하게 사용할 수 있게 해준다.

또한 동일한 타입(int)의 매개변수들을 마치 set 메서드와 같이 이름과 함께 설정하기 때문에 가독성이 좋아지는 이점을 얻을 수 있다.



빌더 패턴은 (파이썬과 스칼라에 있는) 명명된 선택적 매개변수(named optional parameters)를 흉내낸 것이라 한다.



빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다. 

이때 각 계층의 클래스에 관련 빌더를 멤버로 정의한다. 추상 클래스는 추상 빌더를, 구체 클래스(concrete class)는 구체 빌더를 갖게 한다.

책에서는 Pizza 라는 abstract class 를 만들고 abstract Builder 를 만든다. 그리고 구체 클래스인 NyPizza와 Calzone 를 만들어 각각의 클래스에서 구체 빌더를 갖도록 한다. 구체 빌더에서는 각각의 클래스에서 필요로하는 각자의 구체적인 빌더 메서드를 추가로 정의해 사용하도록 했다.



빌더 패턴의 장점은 유연성이다. 빌더 하나로 여러 객체를 순회하면서 만들 수 있고, 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수도 있다.

객체마다 부여되는 일련번호와 같은 특정 필드는 빌더가 알아서 채우도록 할 수도 있다. 

이렇듯 빌더 패턴의 장점과 초반에 이야기한 단점을 고려하여 빌더 패턴을 유용하게 사용할 수 있도록 하자.