# Item 29. 이왕이면 제네릭 타입으로 만들라

### 제네릭이 절실한 강력 후보

```java
public class Stack {
   private Object[] elements;
   private int size = 0;
   private static final int DEFAULT_INITIAL_CAPACITY = 16;

   public Stack() {
       elements = new Object[DEFAULT_INITIAL_CAPACITY];
   }

   public void push(Object e) {
       ensureCapacity();
       elements[size++] = e;
   }

   public Object pop() {
       if (size == 0) {
           throw new EmptyStackException();
       }

       Object result = elements[--size];
       elements[size] = null;
       return result;
   }
}
```

-   클라이언트는 스택에서 꺼낸 객체를 형변환해야 하는데, 이때 런타임 오류가 날 수 있다.
-   일반 클래스를 제네릭 클래스로 만든는 첫 단계는 클래스 선언에 타입 매개변수를 추가하는 것이다.
-   타입 이름으로 보통 E를 사용한다.

```java
public class GenericStack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public GenericStack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY]; // 에러 발생!
        // 실체화 불가 타입으로는 배열을 만들 수 없음.
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        E result = elements[--size];
        elements[size] = null;
        return result;
    }
}
```

-   제네릭은 실체화 불가 타입
-   런타임 정보가 컴파일타임 정보보다 적다.
-   배열 생성 불가 (item 28)

<br/>

### 우회 방법 1

-   배열을 Object으로 생성하고 Generic으로 형 변환

```java
@SuppressWarnings("unchecked")
public Stack(){
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

<br/>

### 우회 방법 2

-   elements의 타입을 Object[]로 선언하고, push, pop 메서드에서 형변환

```java
public class GenericStack<E> {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public GenericStack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        @SuppressWarnings("unchecked")
        E result = (E) elements[--size];
        elements[size] = null;
        return result;
    }
}
```

-   push 에서 E 타입만 넣을 수 있으므로, elements 는 E 로 무조건 타입변환이 가능

<br/>

### 우회 방법 1 vs 2

-   첫번쨰 방법은 elements 배열을 쓰는 곳 여러군데가 있더라도 타입 캐스팅을 생성자에서 한번만 함
    -   두번째 방법은 매번 타입 캐스팅을 해줘야 함
-   첫번쨰 방법이 가독성이 좋음
-   단, 첫번쨰 방법은 \*힙 오염이 생길 수 있음

<br/>

### 힙 오염 (Heap Pollution)

-   매개변수화 타입이 매개변수화 타입이 아닌 것을 참조할 떄 발생하는 현상
-   매개변수화 타입은 **\*타입 소거**가 이루어진다
-   따라서 런타임 시점에는 타입 정보를 모른다
-   하지만 매개변수화 타입이 아닌 배열은 런타임 시점에 타입 정보를 안다
-   따라서 컴파일타임과 런타임의 정보가 달라서, UncheckedWarning 과 ClassCastException 이 발생

**_\*타입소거 : 컴파일 시점(이후)에 타입 정보를 지운다_**
매개변수화 타입 | 소거 후
|---------|-----|
List<String> | List
Map.Entry<String,Long> | Map.Entry
Pair<Long,Long>[] | Pair[]
Comparable<? super Number> | Comparable

<br/>

### 힙 오염이 발생하는 경우

-   raw 타입과 매개변수화 타입을 같이 사용하는 경우

```java
List ln = new ArrayList<Number>();
List ls = new LinkedList<String>();
List<String> list;
list = ln; // unchecked warning + heap pollution
list = ls; // unchecked warning + NO heap pollution
```

-   unchecked 캐스팅을 하는 경우

```java
List<? extends Number> ln = new ArrayList<Long>();
List<Short> ls = (List<Short>) ln; // unchecked warning + heap pollution
List<Long>  ll = (List<Long>)  ln; // unchecked warning + NO heap pollution
```

<br/>

### 정리

-   클라이언트에서 직접 형변환해야 하는 경우가 많다면, 제네릭 타입으로 만들자
-   제네릭 타입으로 만들 수 없는 경우라면, 형변환을 해도 안전한지 확인하자
-   안전하다면 @SuppressWarnings("unchecked") 어노테이션을 달아 경고를 숨기자 (item 27)
-   안전하지 않다면, 형변환을 하지 말자
-   제네릭 타입을 쓰면, 컴파일 타임에 오류를 잡을 수 있고, 형변환을 하지 않아도 된다
-   제네릭 타입을 쓰면, 코드도 간결해지고, 가독성도 좋아진다
-   제네릭 타입을 쓰면, 런타임에 ClassCastException 이 발생할 수 있다
-   제네릭 타입을 쓰면, 힙 오염이 발생할 수 있다
-   제네릭 타입을 쓰면, 배열을 만들 수 없다