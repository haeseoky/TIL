# item02.생성자에 매개변수가 많다면 빌더를 고려하라

### 생성자에 매개변수가 많을 때 고려할 수 있는 방안
#### 대안1. 점층적 생성자 패턴 / 생성자 체이닝
정적 팩토리와 생성자에서는 선택적 매개변수가 많을때 적절히 대응하기 어렵다.

아래와 같이 식품 포장의 영양정보를 표현하는 클래스가 있다.
여러가지 값들이 있지만 대다수 제품의 선택항목 값은 0이다.
많은 프로그래머들은 이러한 클래스의 생성자 혹은 정적 팩토리는 점증척 생성자 패턴을 사용한다고 한다.
```java
public class NutritionFacts {
	private final int servingSize;  // (mL, 1회 제공량)     필수
	private final int servings;     // (회, 총 n회 제공량)  필수
	private final int calories;     // (1회 제공량당)       선택
	private final int fat;          // (g/1회 제공량)       선택
	private final int sodium;       // (mg/1회 제공량)      선택
	private final int carbohydrate; // (g/1회 제공량)       선택

	public NutritionFacts(int servingSize, int servings) {
		this(servingSize, servings, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories) {
		this(servingSize, servings, calories, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories, int fat) {
		this(servingSize, servings, calories, fat, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
		this(servingSize, servings, calories, fat, sodium, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
		this.servingSize = servingSize;
		this.servings = servings;
		this.calories = calories;
		this.fat = fat;
		this.sodium = sodium;
		this.carbohydrate = carbohydrate;
	}
}
```
```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```

**점층적 생성자 패턴도 쓸 수는 있지만, 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.**


#### 대안2. 자바빈즈 패턴
자바빈즈 패턴은 매개변수가 없는 생성자로 객체를 만든 후, 세터 메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식이다.
```java
public class NutrutionFacts {
	private final int servingSize;  // (mL, 1회 제공량)     필수
	private final int servings;     // (회, 총 n회 제공량)  필수
	private final int calories;     // (1회 제공량당)       선택
	private final int fat;          // (g/1회 제공량)       선택
	private final int sodium;       // (mg/1회 제공량)      선택
	private final int carbohydrate; // (g/1회 제공량)       선택

	public NutrutionFacts(int servingSize, int servings) {
		this(servingSize, servings, 0);
	}

	public NutrutionFacts(int servingSize, int servings, int calories) {
		this(servingSize, servings, calories, 0);
	}

	public NutrutionFacts(int servingSize, int servings, int calories, int fat) {
		this(servingSize, servings, calories, fat, 0);
	}

	public NutrutionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
		this(servingSize, servings, calories, fat, sodium, 0);
	}

	public NutrutionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
		this.servingSize = servingSize;
		this.servings = servings;
		this.calories = calories;
		this.fat = fat;
		this.sodium = sodium;
		this.carbohydrate = carbohydrate;
	}
}
```
점층적 생성자 패턴의 단점들이 자바빈즈 패턴에서는 더 이상 보이지 않는다.
코드가 길어지긴 했지만 인스턴스를 만들기 쉬워 더 읽기 쉬운 코드가 되었다.

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

하지만 자바빈즈 패턴의 단점은
- 객체 하나를 만들려면 메서드를 여러 개 호출해야 한다.
- 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다.
- 런타임에 문제를 겪는 코드가 물리적으로 멀리 떨어져 있어 디버깅을 하기 어렵다.
- 자바빈즈 패턴에서는 클래스를 **불변으로 만들 수 없다.**

또한, 자바빈즈 패턴을 사용하게되면 필수 값을 set하기 전 사용 될 수 있다. (일관성 문제)
그럼 생성자에 필수값들만 받으면 되지 않나요?
```java
	public NutritionFacts(int servingSize, int servings) {
		this.servingSize = servingSize;
		this.servings = servings;
	}
```
```java
NutritionFacts cocaCola = new NutritionFacts(240, 8);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

그렇다.</br>
하지만 불변(Immutable) 객체로 만들 수 없다는 단점은 여전하다.</br>
그러면 객체를 어떻게 불변 객체로 만들 수 있을까?</br>
freezing이라는 기술을 쓸 수 있다. 하지만 현업에서도 잘 사용되지 않으며, 자바에서는 직접 구현해야 한다.</br>
보통 JS 진영에서 많이 사용하는데
```javascript
'use strict';

const domo = {
  'name': 'kim dong ho',
  'age': 28
};

Object.freeze(domo);

domo.job = ["backend engineer"];

console.info(domo.job);
```
위 코드를 가지고 설명하자면 JS에서는 객체를 선언한뒤 property를 마음대로 추가할 수 있다.</br>
하지만 freeze 메서드로 얼리고나서 property를 추가하려고 하면</br>
`Cannot add property kids, object is not extensible` 에러가 발생한다.

이것을 자바에서 구현하려고 한다면
```java
private boolean frozen = false;

public void setServingSize(int servingSize) {
  checkIfObjectIsFrozen();
  this.servingSize = servingSize;
}

private void freeze() {
  this.frozen = true;
}

private void checkIfObjectIsFrozen() {
  if (this.frozen) {
    throw new IllegalStateException("Object is frozen");
  }
}
```
와 같이 boolean으로 선언된 frozen을 기준으로 체크하면서 변경하려고 할때 변경하지 못하도록 예외를 던지는 방법이 있다.</br>
하지만 이렇게는 사용하지 않는다고 한다.</br>

#### 대안3. 빌더패턴
```java
public class NutritionFacts {
	private int servingSize;  // (mL, 1회 제공량)     필수
	private int servings;     // (회, 총 n회 제공량)  필수
	private int calories;     // (1회 제공량당)       선택
	private int fat;          // (g/1회 제공량)       선택
	private int sodium;       // (mg/1회 제공량)      선택
	private int carbohydrate; // (g/1회 제공량)       선택

	private NutritionFacts(Builder builder) {
		servingSize = builder.servingSize;
		servings = builder.servings;
		calories = builder.calories;
		fat = builder.fat;
		sodium = builder.sodium;
		carbohydrate = builder.carbohydrate;
	}

	public static class Builder {
		// 필수 매개변수
		private final int servingSize;
		private final int servings;

		// 선택 매개변수 - 기본값으로 초기화
		private int calories = 0;
		private int fat = 0;
		private int sodium = 0;
		private int carbohydrate = 0;

		public Builder(int servingSize, int servings) {
			this.servingSize = servingSize;
			this.servings = servings;
		}

		public Builder calories(int val) {
			calories = val;
			return this;
		}

		public Builder fat(int val) {
			fat = val;
			return this;
		}

		public Builder sodium(int val) {
			sodium = val;
			return this;
		}

		public Builder carbohydrate(int val) {
			carbohydrate = val;
			return this;
		}

		public NutritionFacts build() {
			return new NutritionFacts(this);
		}
	}
}
```
```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
				.calories(100).sodium(35).carbohydrate(27).build();
```

이 클라이언트 코드는 쓰기 쉽고, 읽기 쉽다.</br>
빌더 패턴은 명명된 선택적 매개변수를 흉내 낸 것이다.

요즘은 대부분 lombok을 사용하여 `@Builder`를 사용한다.</br>
`@Builder` 어노테이션을 붙이면 컴파일 할 때 어노테이션 정보를 읽어 클래스 코드를 조작하여 Builder를 생성해준다.</br>

`@Builder` 어노테이션을 사용하였을때 단점
- lombok을 사용한 Builder는 모든 파라미터를 받는 생성자를 만든다.</br>
이러한 경우는 `@AllArgsConstructor(access = AccessLevel.PRIVATE)`을 선언해주면 클래스 내부의 빌더만 모든 파라미터를 받는 생성자를 사용할 수 있다.
- 필수 값을 지정할 수 없다 -> 필수 값을 지정하려면 `@Builder` 어노테이션 대신, 직접 Builder를 구현해야 한다.

위 경우가 아니라면 어노테이션을 사용하여 상당한 부분의 코드를 줄일 수 있다.

#### Builder를 계층 구조에서 사용하는 방법
```java
public abstract class Pizza {
	public enum Topping = { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE };
	final Set<Topping> toppings;

	abstract static class Builder<T extends Builder<T>> {
		EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

		public T addTopping(Topping topping) {
			toppings.add(Objects.requireNonNull(topping));
			return self();
		}

		abstract Pizza build();

		// 하위 클래스는 이 메서드를 재정의(overriding)하여
		// "this"를 반환하도록 해야 한다.
		protected abstract T self();
	}

	Pizza(Builder<?> builder) {
		toppings = builder.toppings.clone(); // 아이템 50 참조
	}
}

public class NyPizza extends Pizza {
	public enum Size { SMALL, MEDIUM, LARGE }
	private final Size size;
	
	public static class Builder extends Pizza.Builder<Builder> {
		private final Size size;
		
		public Builder(Size size) {
			this.size = Objects.requireNonNull(size);
		}
		
		@Override public NyPizza build() {
			return new NyPizza(this);
		}
		
		@Override protected Builder self() { return this; }
	}
	
	private NyPizza(Builder builder) {
		super(builder);
		size = builder.size;
	}
}

public class Calzone extends Pizza {
	private final boolean sauceInside;

	public static class Builder extends Pizza.Builder<Builder> {
		private boolean sauceInside = false; // 기본값

		public Builder sauceInside() {
			sauceInside = true;
			return this;
		}

		@Override
		public Calzone build() {
			return new Calzone(this);
		}

		@Override
		protected Builder self() {
			return this;
		}
	}

	private Calzone(Builder builder) {
		super(builder);
		sauceInside = builder.sauceInside;
	}
}
```

```java
NyPizza pizza = new NyPizza.Builder(SMALL)
				.addTopping(SAUSAGE).addTopping(ONION).build();
Calzone calzone = new Calzone.Builder()
        .addTopping(HAM).sauceInside().build();
```
**추상 메서드의 self 덕분에 Builder를 사용하는 곳에서 형변환이 필요하지 않다.**
self타입이 없는 자바를 위한 이 우회 방법을 시뮬레이트한 셀프 타입(simulated self-type) 관용구라 한다.

#### 핵심정리
생성자나 정적 팩토리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는게 더 낫다.</br>
매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다.</br>
빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.