# Item 20. 추상 클래스보다는 인터페이스를 우선하라

1. **다중 구현용 타입으로는 인터페이스가 가장 적합**
    - 일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합합니다. 인터페이스는 재사용성, 유연성 그리고 다형성 측면에서 우수합니다.

```java
interface Animal {
    void eat();
    void sleep();
}

class Dog implements Animal {
    @Override
    public void eat() {
        System.out.println("Dog is eating.");
    }

    @Override
    public void sleep() {
        System.out.println("Dog is sleeping.");
    }
}

class Cat implements Animal {
    @Override
    public void eat() {
        System.out.println("Cat is eating.");
    }

    @Override
    public void sleep() {
        System.out.println("Cat is sleeping.");
    }
}
```

2. **복잡한 인터페이스와 골격 구현**
    - 복잡한 인터페이스의 경우, 구현하는 수고를 덜어주는 골격 구현(skeletal implementation)을 함께 제공하는 것이 좋습니다. 디폴트 메소드를 사용하지 않고 추상 골격 구현 클래스(AbstractCharacter)를 구현하여 중복을 없앨 수 있습니다.

```java
abstract class AbstractAnimal implements Animal {
    @Override
    public void eat() {
        System.out.println("The animal is eating.");
    }

    // Sleep method is left to be implemented by subclasses
}

class Dog extends AbstractAnimal {
    @Override
    public void sleep() {
        System.out.println("Dog is sleeping.");
    }
}

class Cat extends AbstractAnimal {
    @Override
    public void sleep() {
        System.out.println("Cat is sleeping.");
    }
}
```

3. **추상 클래스의 단점**
    - 추상 클래스는 다중 상속이 되지 않기 때문에 확장에 용이하지 않습니다. 이는 상위/하위 구조만 존재할 수 있습니다.

```java
// 예시 3: 추상 클래스의 단점
abstract class AbstractParent {
    abstract void method();
}

// Cannot extend more than one abstract class
//class Child extends AbstractParent, AnotherAbstractParent { }
```

4. **인터페이스가 확장에 용이한 이유**
    - 기존 레거시 클래스에도 손쉽게 확장 인터페이스를 구현할 수 있습니다. 자바의 표준 라이브러리의 확장은 인터페이스를 통해 이루어지며, 인터페이스는 Mixin 정의가 가능합니다. 이를 통해 주 타입 형태에 상황에 따라 선택적인 기능을 추가할 수 있습니다. 또한, 인터페이스는 계층구조가 없는 타입 프레임워크를 만들 수 있습니다.

```java
// 예시 4: 인터페이스 확장의 용이성
interface LegacyInterface {
    void legacyMethod();
}

class LegacyClass implements LegacyInterface {
    public void legacyMethod() {
        System.out.println("Implemented in legacy class");
    }
}
```

5. **default 메소드를 Interface에서 사용하려면 잘 사용하라**

    - 개념적으로 하나로 확정하기 어려운 기능은 default 메소드를 제공하지 않는 것이 좋습니다. 공통으로 사용되는 기능이 하나의 구현으로 책임질 상황에서만 사용하며, 이를 주석이나 명세로 다른 사람에게 전달될 수 있어야 합니다.
    - Java 8부터 인터페이스에 디폴트 메소드를 추가할 수 있으나, 여전히 모든 메소드에 대해 디폴트 구현을 제공하는 것은 적절하지 않을 수 있으며, 이 경우에는 추상 클래스를 활용하여 중복을 최소화하는 것이 좋습니다.
    - 인터페이스는 기능에 대한 구현보다는, 기능에 대한 '선언'에 초점을 맞추어서 사용합니다.

```java
// 예시 5: Interface에서의 default 메소드 사용
interface DefaultMethodInterface {
    default void defaultMethod() {
        System.out.println("Default method");
    }
}

class DefaultMethodClass implements DefaultMethodInterface {
    // No need to override defaultMethod()
}
```

6. **추상골격 구현과 인터페이스를 같이 고려하라**
    - 디자인 패턴으로는 템플릿 메소드 패턴이 있습니다. 추상 클래스와 인터페이스를 둘 다 사용하는 것이 좋습니다.

```java
// 예시 6: 추상골격 구현과 인터페이스
interface InterfaceA {
    void methodA();
}

abstract class AbstractClassA implements InterfaceA {
    abstract void methodB();
}

class ConcreteClassA extends AbstractClassA {
    public void methodA() {
        System.out.println("method A");
    }

    public void methodB() {
        System.out.println("method B");
    }
}
```

<br/>

### 결론

-   인터페이스 기반의 확장을 사용하되 골격 클래스는 추상 클래스로 구현하는 방식을 권장합니다. 이렇게 하면 구현 클래스는 필요한 부분만 구현하면 되므로 더욱 간결하고 명확한 코드를 작성할 수 있습니다.

-   기반 메소드는 인터페이스로 정의하고, 상황에 따라 선택적으로 default 메소드를 사용합니다.

-   추상 클래스에서 인터페이스의 메소드를 사용할 때는 abstract 메소드로 정의합니다. 이 방식을 통해 추상 클래스는 인터페이스를 완전히 구현하지 않고 하위 클래스에게 구현을 위임할 수 있습니다.

-   기반 메소드가 아니지만 필요한 메소드는 추상 클래스의 메소드로 정의합니다. 이는 공통적인 기능을 추상 클래스에서 제공하면서, 필요에 따라 하위 클래스에서 이를 재정의할 수 있는 유연성을 제공합니다.

-   추상 골격을 고려해서 확장이 필요하면 **\*Mixin Interface**를 사용합니다. 이를 통해 클래스는 주 타입에 추가적인 기능을 가질 수 있으며, 이는 더욱 풍부한 다형성을 제공합니다.

> [\*기본 인터페이스 메서드를 사용하는 인터페이스를 통해 클래스를 만드는 경우의 기능 혼합](https://learn.microsoft.com/ko-kr/dotnet/csharp/advanced-topics/interface-implementation/mixins-with-default-interface-methods)

-   Mixin 인터페이스는 주 타입에 선택적 기능을 제공하기 위해 사용되는 인터페이스
-   Mixin 인터페이스는 주로 구현 클래스가 주 타입 외에 추가적인 인터페이스를 구현할 수 있게 하여 클래스의 기능을 확장하는 데 사용
-   Java에서는 실제로 'mixin'이라는 키워드는 없지만, 인터페이스와 기본 메서드(default methods)를 통해 비슷한 기능을 구현할 수 있음
-   예를 들어, Drawable이라는 mixin 인터페이스를 만들어 '그릴 수 있는' 기능을 제공할 수 있습니다. 아래는 이에 대한 간단한 예시 코드:
    -   Circle 클래스는 원래 '그릴 수 있는' 기능이 없지만, Drawable 인터페이스를 확장하여 DrawableCircle라는 새 클래스를 만들어 '그릴 수 있는' 기능을 추가

```java
// Mixin Interface
public interface Drawable {
    void draw();
}

// Concrete class
public class Circle {
    private int radius;

    // Constructor and other methods...
}

// Now we want to make Circle drawable without changing its class definition.
public class DrawableCircle extends Circle implements Drawable {
    public DrawableCircle(int radius) {
        super(radius);
    }

    @Override
    public void draw() {
        System.out.println("Drawing a circle");
    }
}
```