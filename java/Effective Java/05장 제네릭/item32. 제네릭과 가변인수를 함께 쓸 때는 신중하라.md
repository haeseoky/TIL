### ☁️ 가변인수 메서드
가변인수(varargs) 메서드란, 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 하는 것이다. 제네릭과 같이 자바 5에서 추가되었다.

```java
void sum(String...str) {  // ... : 가변인자
	for(String a:str)
    	System.out.println(a);
}
```

가변인수 메서드를 호출하면, **가변인수를 담기 위한 배열**이 자동으로 만들어진다. 

따라서 제네릭과 가변인수를 함께 사용하게 되면 제네릭 배열이 생성되고, 아이템 28에서 보았던 힙 오염이 발생하며 아래와 같은 컴파일 경고를 내뱉게 된다.

> warning : [unchecked] Possible heap pollution from
parameterized vararg type `List<String>`

#### 힙 오염 예시

매개변수화 타입의 변수가 타입이 다른 객체를 참조하면, **힙 오염**이 발생한다. 아래 예제에서는 `get` 에서 컴파일러가 보이지 않게 형변환을 하기 때문에, `ClassCastException` 이 발생하게 된다.

```java
public class Dangerous {
    static void dangerous(List<String>... stringLists) {   //가변인수 메서드
        List<Integer> intList = List.of(42);
        Object[] objects = stringLists;
        objects[0] = intList; // 힙 오염 발생
        String s = stringLists[0].get(0); // ClassCastException
    }

    public static void main(String[] args) {
        dangerous(List.of("There be dragons!"));
    }
}
```

결론적으로 이처럼 타입 안전성이 깨지기 때문에, **제네릭 varargs 매개변수에 값을 저장하는 것은 안전하지 않다.**

> Q.제네릭 배열을 직접 선언하는 것은 허용하지 않으면서, 제네릭 varargs 매개변수를 받는 메서드의 선언은 허용한 이유?

하지만 제네릭이나 매개변수화 타입의 varagrs 매개변수를 받는 메서드는 실무에서 매우 유용하기 때문에, 모순을 허용하였다. 그리고 타입 안전함이 확실히 보장된다면 `@SafeVarags` 를 통해 아예 컴파일러가 경고를 하지 않게 만들 수 있다. 
```java
Arrays.asList(T... a),
Collections.addAll(Collection<? super T> c, T... elements)
Enumset.of(E first, E...rest)
```

![](https://velog.velcdn.com/images/semi-cloud/post/dc94a765-48bf-4c67-aac3-14fbfd39a999/image.png)


### ☁️ @SafeVarargs

#### 자바 7 이전

제네릭 가변인수 메서드를 사용함으로써 발생하는 경고에 대해 호출하는 곳 마다`@SuppresssWarnings("unchecked")` 를 달아 숨겨야 한다는 단점이 있었다.
#### 자바 7 이후
 `@SafeVarargs` 을 통해, **메서드 작성자가 그 메서드가 타입 안전한지 보장**할 수 있게 되었다. (컴파일러 경고를 없앨 수 있다)
 
그러므로 힙 오염이 발생하지 않는 등 안전하다면 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 `@SafeVarargs` 를 달자.

그러면 어떤 경우 안전하다고 판단할 수 있는가?

**메서드 안전성을 알아보는 방법**

1. varargs 매개변수 배열에 아무것도 저장하지 않는다.
2. varargs 매개변수 배열 혹은 복제본의 참조가 밖으로 노출되지 않는다. (신뢰할 수 없는 코드가 배열에 접근하지 않는다) 노출되는 경우,  아무것도 저장하지 않아도 타입 안전성이 깨질 수 있기 때문이다.


> 🔖 **참고사항**
`@SafeVarargs` 애너테이션은 재정의할 수 없는 메서드에만 달아야 한다. 자바 8에서는 오직 정적 메서드와 `final` 인스턴스 메서드에만 붙일 수 있고, 자바 9부터는 `private `인스턴스 메서드에도 허용된다.

### ☁️ 제네릭 varargs 매개변수를 안전하지 않게 사용하는 경우

제네릭 `varargs` 매개변수 배열에 값을 저장하지 않아도,  **다른 메서드가 접근하도록 허용**하는것 자체만으로도, 의도치 않은 힙 오염이 발생할 수 있으므로 안전하지 않다. 

```java
public class PickTwo {
    static <T> T[] toArray(T... args) {  // 자신의 제네릭 매개변수 배열의 참조를 노출
        return args;
    }

    static <T> T[] pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return toArray(a, b);
            case 1: return toArray(a, c);
            case 2: return toArray(b, c);
        }
        throw new AssertionError(); 
    }

    public static void main(String[] args) { 
        String[] attributes = pickTwo("좋은", "빠른", "저렴한");
        System.out.println(Arrays.toString(attributes));
    }
}
```
컴파일러는 toArray에 넘길 T 인스턴스 2개를 담을 varargs 매개변수 배열을 만드는 코드를 생성하며, 이 때 배열의 타입은 `Object[]`이다.

하지만, 컴파일러는 pickTwo의 반환값을 attributes에 저장하기 위해 `String[]`으로 형변환하는 코드를 자동 생성하게 되고, `Object[]` 는 `String[]` 의 하위 타입이 아니므로 이 형변환은 실패하게 되기 때문에, `ClassCastException` 이 발생한다.

따라서, `@SafeVaragrs`가 붙어있는 곳이나 `varargs` 를 인자로 받지 않는 일반 메서드에 넘기는 것은 안전하다.

> 🔖 **형변환 원칙**
형변환은 상속 관계에서 자동으로 일어나며, **자식 타입은 부모 타입으로 형변환이 가능**하다.(반대는 X)

### ☁️ 제네릭 varargs 매개변수를 안전하게 사용하는 경우

가변 인수 메서드를 안전하게 작성함이 불가능하다면(`@SafeVarargs`를 붙일 수 없다면), 아예 가변인수 대신 List 를 사용하면 된다. 오히려 배열 없이 제네릭만 사용하기 때문에 컴파일러가 이 메서드의 **타입 안전성을 검증**할 수 있어 더욱 안전해진다.

```java
@SafeVarargs   
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
         result.addAll(list);
    return result;
}
```
#### 바꾼 예시
```java
public class FlattenWithList {
    static <T> List<T> flatten(List<List<? extends T>> lists) {
    	...
    }

    public static void main(String[] args) {
        List<Integer> flatList = flatten(List.of(
                List.of(1, 2), List.of(3, 4, 5), List.of(6,7)));
        System.out.println(flatList);
    }
}
```

```java
 static <T> List<T> pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return List.of(a, b);
            case 1: return List.of(a, c);
            case 2: return List.of(b, c);
        }
        throw new AssertionError(); 
    }
```

> 📚 **핵심 정리**
가변인수와 제네릭은 궁합이 좋지 않다. 가변인수 기능은 배열을 노출하여 추상화가 완벽하지 못하고, 배열과 제네릭의 타입 규칙이 서로 다르기 때문이다. 제네릭 varargs 매개변수는 타입 안전하지는 않지만, 허용된다. 메서드에 제네릭(혹은 매개변수화된) varargs 매개변수를 사용하고자 한다면, 그 메서드가 타입 안전한지 확인한 다음 @SafeVarargs 애너테이션을 달아 사용하는 데 불편함이 없게끔 하자.