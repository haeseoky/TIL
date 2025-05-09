### ☁️ 태그 달린 클래스
태그 달린 클래스란, **하나의 클래스가 두 가지 이상의 의미를 표현** 가능할 때, 그중 현재 표현하는 의미를 태그 값으로 알려주는 클래스이다. 아래 예시를 보자.


```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };  // TAG 

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

####  태그 달린 클래스 단점

1. 열거 타입 선언, 태그 필드, `switch` 문 등 쓸데없는 코드로 가독성이 떨어진다.
2. 다른 의미를 위한 코드도 해당하지 않는데 항상 존재하니 메모리를 많이 먹는다.
3. 필드들을 불변을 위해 `final` 로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화해야한다.
4. 또 다른 의미를 추가하려면 `switch` 문 코드를 수정해야 한다.
5. 인스턴스의 타입만으로는 현재 나타내는 의미를 알 길이 전혀 없다.

> 즉, 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다.

### ☁️ 클래스 계층 구조
이러한 문제를, 클래스 계층구조를 활용하는 서브타이핑 (subtyping)을 통해 해결할 수 있다. 클래스 계층 구조는, **추상 클래스와 추상 메서드**를 통해 **타입 하나로 다양한 의미의 객체를 표현**할 수 있게 해준다. 


1.  계층 구조의 루트가 될 추상 클래스 정의한다.
```java
abstract class Figure {
    abstract double area();
}
```
+ 태그 값에 따라 **동작이 달라지는 메서드**들을 루트 클래스의 **추상 메서드**로 선언
+ 태그 값에 상관없이 **동작이 일정한 메서드와 공통 메서드**들을 루트 클래스에 **일반 메서드**로 선언


2. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 지정한다.

```java
class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override double area() { return Math.PI * (radius * radius); }
}
```

```java
class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    @Override double area() { return length * width; }
}
```
+ 루트 클래스가 정의한 추상 메서드를 각자 의미에 맞게 구현

#### 클래스 계층 구조 장점

1. 각 의미를 독립된 클래스에 담아 관련 없던 데이터 필드들을 제거했다.
2. 남아 있는 필드들은 모두 `final` 로 선언해 불변을 보장할 수 있다.
3. 각 클래스의 생성자가 모든 필드를 남김없이 초기화하고 추상 메서드를 모두 구현했는지 컴파일러 단에서 확인할 수 있다.
3. 루트 클래스의 코드를 건드리지 않고 독립적으로 계층 구조를 확장할 수 있다.