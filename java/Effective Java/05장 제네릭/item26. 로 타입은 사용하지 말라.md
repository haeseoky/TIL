
- 클래스와 인터페이스 선언에 타입 매개변수(type parameter)가 쓰이면, 이를 **제네릭 클래스** 혹은 **제네릭 인터페이스**라 한다.
- 각각의 제네릭 타입은 일련의 매개변수화 타입(parameterized type)을 정의한다.
- 제네릭 타입을 하나 정의하면 그에 딸린 로 타입(raw type)도 함께 정의된다
	- List의 로 타입은 List
	- 제네릭 정보가 모두 지워진 것 처럼 동작, 하위호환을 위한 동작



### 컬렉션의 로 타입 - 컴파일간 문제를 파악할 수 없음
```java
class RawCollection {
    private final Collection stamps = null;

    void sample() {
        stamps.add(new Coin()); // "unchecked call" 경고
    }

    private static class Coin {

    }
}
```




- List와 같은 로 타입은 사용해서는 안 된다. 하지만 `List<Object>`처럼 임의 객체를 허용하는 매개변수화 타입은 괜찮다.
- List는 제네릭 타입에서 완전히 발을 뺀 것이고, `List<Object>`는 모든 타입을 허용한다는 의사를 컴파일러에 명확히 전달한 것이다.
- 매개변수로 List를 받는 메서드에 List을 넘길 수는 있지만, `List<Object>`를 받는 메서드에는 넘길 수 없다.
    - 이는 제네릭의 하위 타입 규칙 때문이다.
    - `List<String>`은 로 타입인 List의 하위 타입이지만, `List<Object>`의 하위 타입은 아니다.
    - `List<Object>` 같은 매개변수화 타입을 사용할 때와 달리 List 같은 로 타입을 사용하면 타입 안정성을 잃게 된다.



### 로 타입에은 타입 불변성을 해침 - 대신 비한정적 와일드카드 타입

관련 링크
https://stackoverflow.com/questions/29342117/what-is-the-purpose-of-list-if-one-can-only-insert-a-null-value

```java
public static void printList(List<?> list) {
    for (Object elem: list)
        System.out.print(elem + " ");
    System.out.println();
    // List에 값을 넣을때 타입 안정성을 해칠 수 있음 - 어떤 Object인지 알 수가 없음, 하지만 Null은 가능
}
```

-  컬렉션의 불변식을 훼손하지 못하게 막아준다.
-  비한정적 와일드카드를 사용한 코드를 보면 타입객체의 타입도 전혀 알 수 없다. - 타입 불변성 유지



## 로 타입이 사용가능한 예외
단순히 비교하는 부분에만 로타입을 허용

### 1. class 리터럴에는 로 타입 사용
List.class O
List<String>.class X

### 2. Instance of로 타입 사용시 로 타입 사용가능
아래 경우는 Set Type인지 확인
```java
if (o instanceof Set) {
    Set<?> s = (Set<?>) o;
}
```