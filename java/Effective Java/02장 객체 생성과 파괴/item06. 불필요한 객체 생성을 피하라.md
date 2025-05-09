**똑같은 기능의 객체를 매번 생성하기 보다는**, **객체 하나를 재사용**하는 것이 좋은 경우가 많다. 예를 들어, 불변 객체는 언제든지 재사용이 가능하다.

불필요한 객체를 만들어내는 예시들을 살펴보자.

### ☁️ 문자열

#### 1. String 객체
```java
String s = new String("bikini");
```
문자열은 불변의 속성을 가진다. 따라서, `new` 를 통해 새로운 값을 할당할 때마다 **새로운 String 인스턴스가 힙 영역에 생성**된다. 

```java
s = s + "add";
```
따라서 다음과 같이 문자열을 추가하면, 기존의 `bikini` 문자열 값으로 할당 되어 있던 메모리 영역은 참조가 떨어져 나갔으니 GC(Garbage Collector)의 대상이 되고, `bikini add` 라는 값을 가진 새로운 힙 객체가 생성이 된다.


#### 2. String literal

```java
String s = "binkini";
```
매번 새로운 인스턴스를 만드는 대신, **하나의 String 인스턴스를 재사용**하는 방식이다.

`String literal` 로 생성하면 해당 String 값은 `Heap` 영역 내 **String Constant Pool**에 하나만 저장이 되므로, 모든 코드가 같은 객체를 재사용함이 보장된다. 

![](https://velog.velcdn.com/images/semi-cloud/post/2c6db71b-aeb4-4b5a-ae80-de8e4929a3f5/image.png)

https://codepumpkin.com/string-pool-java/

> 참고로 자바 7까지는 상수 풀을 `Permanent` 라는 **정적 영역**에서 관리했었다. 하지만 할당 받은 메모리를 늘릴수가 없어 관리하기 어렵다는 점, `StackOverFlow` 가 발생할 여지 때문에 자바 8에서 **상수 풀을 힙 영역 내부**로 옮겼다고 한다. 

### ☁️(추가) String 문자열 연산자 / StringBuilder / String Buffer
참고로, 문자열 추가/삭제/수정 등 연산이 빈번하게 일어나는 경우 위처럼 **힙 영역에 많은 임시 가비지가 생성**되는 문제는 `StringBuilder` 와 `StringBuffer` 를 통해 해결할 수 있다. 

둘 다 연산이 수행되면 새로운 객체를 생성하는 것이 아닌, 기존 메모리에 추가하는 방식으로 동작한다.

1. `StringBuilder` : 단일 스레드 환경에서 권장, 동기화 지원 X
2. `StringBuffer` : `Synchronized` 키워드를 통해 멀티 스레드 환경에서 동기화를 지원

또한 이펙티브 자바 아이템 63에 따르면(문자열 연결은 느리니 주의하라), 해당 방식을 사용하는 것이 `+` 보다 문자열 `concat` 연산에서 빠른 성능을 보여준다고 한다.

> 하지만 실제로 자바가 버전업 되면서 해당 문제들이 해결이 되면서 속도가 비슷해졌는데, 아래를 통해 자세히 알아보자.

우선 JDK 버전과 관계없이 한 줄에서 상수 `String` 끼리만 더하는 것은, 컴파일 과정에서 모두 하나의 합쳐진 문자열로 바꿔준다. 
```java
String a= "a" + "b" + "c"; 
String = "abc" // 컴파일
```

#### JDK 8 이전

`JDK1.5 - 8` 까지는, 컴파일 단계에서 `String` 객체를 사용하더라도 `StringBuilder` 로 컴파일 되도록 최적화가 되었다. 


```java
String a = str1 + str2 + str3
String a = new StringBuilder(String.valueOf(str1)).append(str2).append(str3).toString(); // 컴파일 
```

하지만 한줄에 한해서는 큰 속도 차이가 없지만, 반복문 내부에서는 상황이 달라진다.

```java
for(int i = 0; i < 100000; i++) {
    s += value;
}
```

`+` 를 썼을 때는 반복문 내부에서 매번 `StringBuilder` 객체를 생성해서 `append` 한 이후에 다시 `toString` 을 통해 문자열로 변환하는 작업을 수행해나가기 때문에 `StringBuilder` 보다 속도가 느려지기 때문이다.


2. JDK 9 이후

`Java9` 이후부터는 `InvokeDynamic` 을 사용하여,  `StringConcatFactory` 클래스의 `makeConcatWithConstants` 라는 메서드를 단일 호출하는 방식으로 최적화되었다. 따라서 `+` 나 `StringBuilder` 모두 속도 측면에서는 똑같아진다.

https://www.baeldung.com/java-string-concatenation-invoke-dynamic
https://gist.github.com/benelog/b81b4434fb8f2220cd0e900be1634753
https://june0122.tistory.com/2
https://june0122.tistory.com/2



## 객체 재사용 예시

### ☁️ 정적 팩터리 메서드
**정적 팩터리 메서드**를 제공하는 불변 클래스에서는, 해당 메서드를 활용하면 **불필요한 객체 생성을 피할 수 있다.** 생성자는 호출할 때마다 새로운 객체를 만들지만, 팩터리 메서드는 하나만 생성해두고 재사용이 가능하기 때문이다.

모든 래핑 클래스의 `valueOf()` 메서드는 객체를 재사용하는 좋은 예시이다.

#### Boolean 클래스

![](https://velog.velcdn.com/images/semi-cloud/post/0c959e7a-88f8-422e-8964-8efab7b3a702/image.png)

![](https://velog.velcdn.com/images/semi-cloud/post/a9da9bbd-e673-4760-be62-331e7945beef/image.png)

`Boolean(String)` 생성자를 이용하는 방식은 자바 9부터 Deprecated API로 되었다. 따라서 `valueOf(String)` 팩터리 메서드를 사용하는 것이 좋다.


### ☁️ 생성 비용이 비싼 객체 : 캐싱
**생성 비용이 아주 비싼 객체가 반복해서 필요하다면, 캐싱해서 재사용** 하는 방법도 존재한다.

```java
static boolean isRomanNumeral(String s){
	return s.matches("^(?=/)M*(C[MD]|D?C{0,3})");
}
```
`String.matches` 메서드 내부에서는 생성 비용이 높은 정규표현식용 `Pattern` 인스턴스가 한번 쓰고 버려지기 때문에, 성능이 중요한 상황에서는 반복해 사용하기에 적합하지 않다. 

#### 성능 개선
```java
public class RomannNumerals{
  private static final Pattern ROMAN= Pattern.compile("^(?=/)M*(C[MD]|D?C{0,3})");

  static boolean isRomanNumeral(String s){
      return ROMAN.matcher(s).matches();
  }
}
```
필요한 정규표현식을 표현하는 `Pattern` 인스턴스를 **클래스 초기화(정적 초기화) 과정에서 직접 생성해 캐싱**해두면, 나중에 메서드가 호출될때마다 인스턴스를 재사용 할 수 있다.

### ☁️ 객체 재사용이 명확하지 않은 경우 : 어댑터
`Map` 인터페이스의 `keySet` 은  맵 객체 안의 키를 전부 담은 `Set` 뷰를 반환하는 일종의 어댑터이다. 

직관적으로 봤을때는, 새로운 객체를 생성해내는 것처럼 보인다.

하지만 어댑터와 같은 경우 실제 작업은 뒷단 객체에 위임하고, 자신은 제2의 인터페이스 역할을 해주는 객체이므로 뒷단 객체 당 어댑터(뷰)가 하나씩만 만들어지면 되는게 맞다.

그리고 실제로도, `Map` 은 같은 `keySet` 을 재사용해서 반환한다. 어짜피 모두 같은 Map 인스턴스를 대변하고 있기 때문에 뷰 객체가 하나만 존재해도 되는 것이다.

```java
public abstract class AbstractMap<K,V> implements Map<K,V> {
    // Views
    // Each of these fields are initialized to contain an instance of the appropriate view the first time this view is requested. The views are stateless, so there's no reason to create more than one of each.

    transient Set<K> keySet;
    transient Collection<V> values;
}

public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    
    public Set<K> keySet() {
        Set<K> ks = keySet;
        if (ks == null) {
            ks = new KeySet();
            keySet = ks;
        }
        return ks;
    }
 }
 ```

### ☁️ 오토박싱(auto boxing)

오토박싱이란, `JDK 1.5` 부터 도입되었으며 기본 타입과 박싱된 기본 타입을 섞어 쓸때 자동으로 상호 변환해주는 기술이다. 

> 기본 타입들은 `Stack Memory` 에 저장되며 접근이 매우 빠르지만, 래퍼 클래스는 `Heap Memory` 에 오버헤드로 인해 접근이 느리다.

스트림을 사용할 때 `.boxed()` 나 `.mapToInt()` 를 호출해주어야 하는 상황을 제외하고는 자동으로 변환되어 알아차리지 못하기 때문에, 잘못 사용하면 불필요한 객체를 생성해내는 문제가 발생한다.

```java
public static long sum(){
	Long sum = 0L;
    
    for(long i=0; i <= Integer.MAX_VALUE; i++)
    	sum += i;
    return sum;
}
```
오토박싱은, 기본형 타입의 값을 래퍼클래스 변수에 할당 할 경우에 일어난다. 
따라서 현재 `Long`으로 선언된 sum 변수에 `long` 타입 값을 더하고 있으니, 불필요한 인스턴스가 `2^31` 개나 만들어지게 되어 성능이 떨어진다.

따라서, 박싱된 기본 타입보다는 **기본 타입을 사용**하여 **의도치 않은 오토박싱이 숨어들지 않도록 주의**하자.

### ☁️ 정리

요즘의 `JVM` 에서는 **가비지 컬렉터가 매우 최적화**되어 있기 때문에, 별다른 일을 하지 않는 작은 객체를 생성하고 회수하는 일이 크게 부담되지 않는다. **따라서 객체 생성은 비싸니 피해야 한다라고 오해하면 안된다.**

> 참고: 가비지 컬렉션 성능 높이기
https://mangkyu.tistory.com/120

또한, 무거운 객체가 아니라면 객체 생성을 피하고자 객체 풀(pool)을 만들면 안된다. DB 연결과 같은 경우 생성 비용이 너무 비싸니 풀을 만들어 재사용 하는 것이 좋지만, 일반적으로 자체적으로 생성한 객체 풀은 오히려  메모리 사용량을 늘리고 성능을 떨어뜨린다.