비검사 경고란, 컴파일 타임에 컴파일러가 해당 선언에서 타입 매개변수를 파악할 수 없어 발생하는 컴파일러 경고를 의미한다.
제네릭을 사용하면서 마주칠 수 있는 비검사 경고들은 쉽게 제거 가능하기 때문에, **할 수 있는 한 모든 비검사 경고를 제거**하는 것이 좋다. 
> 비검사 경고, 비검사 메서드 호출 경고, 비검사 매개변수화 가변인수 타입 경고, 비검사 변환 경고 등..


### ☁️ 비검사 경고 제거 방법
#### 1.  다이아몬드 연산자

```java
Set<Lark> exaltation = new HashSet();
```

javac 명령줄 인수에 `-Xlint:uncheck` 옵션을 추가하면은 컴파일러가 어디가 잘못되었는지 알려준다. 

```
Venery.java:4: warning: [unchecked] unchecked conversion Set<Lark > exaltation = new HashSet();
  required: Set<Lark >
  found: HashSet
```

참고로, 자바 7에서부터 지원하는 **다이아몬드 연산자**( `<>` )을 사용하면 컴파일러가 올바른 실제 타입 매개변수를 추론해주기 때문에 직접 타입 매개변수를 명시하지 않아도 된다.
  
```java
Set<Lark> exaltation = new HashSet<>();
```
  
### ☁️ 비검사 경고를 제거할 수 없는 경우
####  @SuppressWarnings("unchecked")
경고를 제거할 수는 없지만, **타입이 안전하다고 확신**할 수 있다면 `@SuppressWarnings("unchecked")` 어노테이션을 통해 경고를 숨길 수 있다.
  
```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```
`@Target` 에서 볼 수 있듯이 거의 모든 곳에 붙일 수 있지만 **가능한 한 좁은 범위에 적용**하는 것이 좋다. 타입 안전성을 검증하지 않았다면, 경고 없이 컴파일 되지만 런타임에는 여전히 `ClassCastException` 을 던지기 때문이다.
 
만약 한줄이 넘는 메서드나 생성자에 달렸다면, 아래의 예제와 같이 지역변수 선언 쪽으로 옮기는 것을 권장한다.
  
#### 예제
실제 `Arrays.copyOf` 메서드에서는, 지역변수로 범위를 좁게 설정하고 있다.            

![](https://velog.velcdn.com/images/semi-cloud/post/338c0d1c-2ab5-41b7-aeb9-143f2a6ab1cd/image.png)
  
마지막으로, **해당 어노테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.** 다른 사람이 코드를 이해하는데 도움이 되며, 코드를 잘못 수정하여 타입 안전성을 잃는 상황을 줄여주기 때문이다.