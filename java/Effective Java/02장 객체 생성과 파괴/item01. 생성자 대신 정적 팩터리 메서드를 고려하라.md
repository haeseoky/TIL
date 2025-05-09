# item1. 생성자 대신 정적 팩터리 메서드를 고려하라
### ☁️ 정적 팩터리 메서드란
클라이언트가 **클래스의 인스턴스**를 얻는 수단은 다음의 두가지가 존재한다.


1. public 생성자
2. 정적 팩터리 메서드(static factory method)


정적 팩터리 메서드는 **클래스의 인스턴스를 단순히 반환**하기 때문에, 똑같이 객체를 생성하는 디자인 패턴의 팩터리 메서드(Factory method)와는 다르다는 것에 주의하자.


### ☁️ 정적 팩터리 메서드 장점
그렇다면 정적 팩터리 메서드를 사용함으로써 얻을 수 있는 장점에는 무엇이 있을까? 

**1. 이름을 가질 수 있다.**

정적 팩터리는 이름을 잘 짓는다면 **반환될 객체의 특성을 쉽게 묘사 가능**하다. 하나의 시그니처로는 생성자를 하나만 만들 수 있기 때문에, 만약 매개변수들의 순서를 다르게 한 생성자를 추가해나가려고 한다면 그것은 절대 좋은 생각이 아니다.

예를 들어, 아래의 두 가지 선택지 중에 "값이 소수인 BigInteger 반환" 의 의미가 어디에서 더 잘 드러날까? 

1. 생성자 사용한 경우 : `BigInteger(int, int, Random)`
2. 정적 팩터리 메서드 : `BigInteger.probablePrime`

당연히 2번에서 그 뜻이 명확하게 드러난다. 따라서 시그니처가 같은 여러개 생성자가 필요하다면, 정적 팩터리 메서드로 바꾸고 이름을 차이가 드러나게 지어야 한다.

#### 2. 호출시마다 인스턴스 새로 생성하지 않아도 된다.

`Boolean`과 같이 박싱된 기본 타입 클래스 전부와 `BigInteger` 클래스는 **정적 팩터리 메서드를 활용해 자주 사용되는 인스턴스를 캐싱**해서 불필요한 객체 생성을 피하고 있다.

클라이언트가 모두 같은 인스턴스를 공유하여 메모리 사용량과 가비지 컬렉션 비용이 줄어든다는 장점이 있지만, **불변 객체**가 아니라면 동시성 문제가 발생할 수 있다는 점에 주의하자.

예를 들어, `Boolean` 메서드는 아래와 같이 아예 객체를 생성하지 않는다.
```java
public static Boolean valueOf(boolean b) {
	return b ? Boolean.TRUE : Boolean.FALSE;
}
```

또한 `BigInteger`는 int(4byte), long(8byte) 보다 더 큰 정수를 표현해야 할때, 즉 값 범위에 상한과 하한이 없을 때 사용하는 클래스이다. 아래처럼 자주 사용되는 숫자에 한해서 객체를 미리 생성해두고 반환한다. (캐싱)
```java
public class BigInteger extends Number implements Comparable<BigInteger> {
    ...
    public static final BigInteger ZERO = new BigInteger(new int[0], 0);
    public static final BigInteger ONE = valueOf(1);
    public static final BigInteger TWO = valueOf(2);
    private static final BigInteger NEGATIVE_ONE = valueOf(-1);
    public static final BigInteger TEN = valueOf(10);
    ...
}
```
<img src="https://velog.velcdn.com/images/semi-cloud/post/6cbb15de-134a-4a9e-8866-0c28dbf7ef18/image.png" width=70% height=50%> 

<img src="https://velog.velcdn.com/images/semi-cloud/post/16c1cc78-708d-4e5d-9e79-572781be28d3/image.png" width=70% height=50%> 


> 📚 **플라이웨이트 패턴**
인스턴스를 가능한 한 공유해서 사용함으로써 메모리를 절약하는 패턴이다. 즉, 변하지 않는 공통 부분을 `Flyweight` 란 클래스로 분리하고 `FlyweightFactory` 에서 기존에 저장해둔 `Flyweight` 인스턴스가 있다면, 새로 생성하지 않고 기존 인스턴스를 반환하여 공유하는 형식이다. 

또 다른 예시를 살펴보자. 로또 번호를 생성하는 클래스인데, 미리 생성된 로또 번호 객체의 캐싱을 통해서 새로운 객체 생성의 부담을 덜 수 있다는 장점이 존재한다. 
즉 생성자의 접근 제한자를 `private` 으로 설정함으로써 객체 생성을 정적 팩토리 메서드를 통해서만 가능하도록 제한할 수 있다.
```java
public class LottoNumber {
  private static final int MIN_LOTTO_NUMBER = 1;
  private static final int MAX_LOTTO_NUMBER = 45;

  private static Map<Integer, LottoNumber> lottoNumberCache = new HashMap<>();

  static {
    IntStream.range(MIN_LOTTO_NUMBER, MAX_LOTTO_NUMBER)
                .forEach(i -> lottoNumberCache.put(i, new LottoNumber(i)));
  }

  private int number;

  private LottoNumber(int number) {
    this.number = number;
  }

  public LottoNumber of(int number) {  // LottoNumber를 반환하는 정적 팩토리 메서드
    return lottoNumberCache.get(number);
  }
  ...
}
```

이처럼 **반복되는 요청에 같은 객체를 반환**하는 식으로, 정적 팩터리 방식의 클래스는 언제 어느 인스터스 살아있게 할지 통제할 수 있으며 이를 **인스턴스 통제 클래스**라고 한다.

> 인스턴스를 통제 가능할 때 장점

1. 클래스를 싱글턴으로 생성할 수 있다.<br>
`private` 으로 생성자를 막고 메서드를 통해 항상 같은 인스턴스를 제공함으로써, 무상태 객체를 만들 수 있다.

2. 클래스를 인스턴스화 불가로 생성할 수 있다.<br>
상속 금지 클래스나 정적 필드 만을 담은 클래스의 인스턴스화를 막기 위해 `private` 으로 생성자를 막으면 된다.

3. 불변 값 클래스에서 동치 인스턴스가 하나임을 보장할 수 있다.<br>
동치 인스턴스란 `a == b` 일 때만 `a.equals(b)` 가 성립함을 의미한다.

3. 열거 타입과 같이 인스턴스가 하나만 만들어짐을 보장할 수 있다.<br>
열거 타입은 인스턴스를 하나만 생성해서 `public static final` 로 공개하므로 생성자를 직접 호출할 수 없다.

<img src="https://velog.velcdn.com/images/semi-cloud/post/c11f7d00-ad3a-4026-8261-7083e80459fa/image.png" width=70% height=50%> 
https://hudi.blog/java-enum/


#### 3. 반환 타입의 하위 타입 객체를 반환할 수 있다.

즉, **반환할 객체의 클래스를 자유롭게 선택 가능하게 함으로써** **유연성을 높일 수 있다.** <br>
예를 들어 아래와 같이, 정적 팩터리 메서드를 통해 `Level.of(점수)` 를 통해 각 점수에 맞는 레벨을 반환해줄 수 있다.
```java
public class Level {

  public static Level of(int score) {
      if (score < 50) {
         return new Basic();  //상위 클래스 
      } else if (score < 80) {
         return new Intermediate();   // 하위 클래스
      } else {
         return new Advanced();    // 하위 클래스
      }
  }
}

private class Intermediate extends Level {

}
```
하지만 정확히 말하면 위와 같은 유연성을 활용하여 `public` 으로 선언되지 않은 클래스의 객체를 반환하는 API를 만들 수 있다.
그리고 이를 통해, 인터페이스를 정적 메서드의 반환 타입으로 사용하는 인터페이스 기반 프레임워크를 만들 수 있다. 구현 클래스를 공개하지 않고도 객체 반환이 가능하기 때문에 API를 작게 유지할 수 있기 때문이다.

1. 자바 8 이전

**인터페이스에 정적 메서드를 선언할 수 없으므로**, 인스턴스화가 불가한 동반 클래스를 두어 내부에 선언하였다.<br>
예를 들어, 수정 불가능한 리스트를 반환하는 기능을 붙인 `UnmodifiableList` 는 `List` 인터페이스 내부에 정적 메서드를 선언 불가능하기 때문에 인스턴스화 불가 클래스인 `Collections` 에 정의해둔 것을 볼 수 있다. 

![](https://velog.velcdn.com/images/semi-cloud/post/546d509f-96c9-4ea0-ba84-011012bb119c/image.png)

내부 클래스여서 직접적으로 구현체를 들여다보지 않아도 되며, 얻은 객체를 인터페이스만으로 다룰 수 있게 되었다.
결론적으로 컬렉션 프레임워크는 해당 45개의 클래스를 공개하지 않고도 정적 메서드를 통해서 사용자들에게 제공하였기 때문에 API의 외견을 훨씬 작게 만들 수 있었다.

2. 자바 8 이후

**인터페이스의 정적 메소드(static)가 가능해졌기 때문에**, 단순히 public 정적 멤버들을 인터페이스 내부에 두면 된다. 참고로 자바 9는 `private` 정적 메서드는 허락하지만, **필드와 멤버 클래스**는 여전히 `public` 만 가능하여 별도의 `package-private` 클래스에 두어야 할 수도 있다.

#### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체 반환할 수 있다. 

즉 **반환 타입의 하위 타입이기만 하면, 어떤 클래스의 객체를 반환하던 상관 없어지는 것**이다. 예를 들어 정적 팩터리만 제공하는 `EnumSet` 클래스는 두 가지 하위 클래스 중 원소의 수에 따라 알맞게 반환해준다.

<img src="https://velog.velcdn.com/images/semi-cloud/post/dbeae20a-016b-4db5-8678-67669de996fb/image.png" width=70% height=50%>


>
1. 원소가 64개 이하 : `RegularEnumset` (long 변수 하나로 관리)
2. 원소가 65개 이상 : `JumboEnumSet` (long 배열로 관리)


#### 5. 정적 팩터리 메서드 작성 시점에, 반환할 객체의 클래스가 존재하지 않아도 된다.

이러한 유연함은, `JDBC` 와 같은 **서비스 제공자 프레임워크**를 만드는 근간이 된다. <br>
서비스 제공자 프레임워크는 구현체들을 클라이언트에 제공하는 역할을 프레임워크가 통제하여, **클라이언트와 구현체를 분리**시킨다.
즉, 미리 결론을 말하자면 `Driver`, `Connection` 인터페이스와 실제 그 인터페이스를 구현하는 구현체 클래스를 분리시켜놓고, 각각 자신의 서비스에 맞는 구현 클래스를 제공하도록 하는 것이다. 


> 📚 **서비스 제공자 프레임워크의 3가지 핵심 컴포넌트**
1. `Service Interface` : 구현체의 동작을 정의 EX) `JDBC CONNECTION`
2. `Provider Registeration API` : 제공자가 구현체를 등록할때 사용 EX) `DriverManager.registerDriver`
3. `Service Access API` : 클라이언트가 서비스의 인스턴스를 얻을 때 사용 EX) `DriverManager.getConnection`

그렇다면 이게 정적 팩터리 메서드와 무슨 관련이 있을까?
```java
String url = "jdbc:mysql://localhost:3306/XX";
String user = "root";
String password = "1234@";
 
try {
    Class.forName("com.mysql.jdbc.Driver");  // 리플렉션  
    Connection conn = DriverManager.getConnection(url, user, password);  // 정적 팩터리 메서드
} catch (ClassNotFoundException e) {
    e.printStackTrace();
} catch (SQLException e) {
    e.printStackTrace();
}
```
+ `Class.forName(driverName)` 
  + [공식 문서](https://docs.oracle.com/javase/7/docs/api/java/sql/Driver.html)를 보면, 클래스 로더를 통해 `Driver` 인터페이스를 상속 받은 하위 클래스를 찾아서 불러온다.
  + 로드가 일어날 때 `static` 필드의 내용이 실행되는 것을 이용해 `DriverManager` 클래스에 자신을 등록한다. 
  
이처럼 JDBC는, 런타임에 DBMS 드라이버를 선택해서 동작한다. 하지만 어느 드라이버를 로드해야 하는지 실행 시점에는 알 수 없기 때문에, 코드 실행 전 리플렉션을 사용해서 동적으로 드라이버를 로드한다. 리플렉션은 구체적인 클래스 타입을 알지 못해도, 이름만 안다면 바이트 코드로 컴파일 된 자바 클래스의 필드와 메서드 정보가 존재하는 `static` 영역에 접근할 수 있기 때문이다.
`DriverManager.getConnection` 에서 만약 특정한 드라이버를 명시해두지 않았다면 기본 구현체를 반환하는 식으로 유연하게 대처할 수 있다. 

### ☁️ 정적 팩터리 메서드 단점
#### 1. 상속을 하려면 하위 클래스의 생성이 불가능하다.
상속을 하기 위해서는, public/protected 생성자가 필요하니 정적 팩터리 메서드만 제공한다면 상속을 이용한 하위 클래스 생성을 하지 못한다. 

#### 2. 프로그래머가 찾기 힘들다.
생성자처럼 API 설명에 드러나지 않아서, 직접 인스턴스화할 방법을 알아내야 한다. 따라서 이런 불편함들을 해소하기 위해, 현재는 정적 팩터리 메서드에 흔히 사용하는 명명 방식들이 정해져 있다.

### ☁️ 정적 팩터리 메서드 명명 규칙
1. `from` : 매개변수 하나 받아 해당 타입의 인스턴스 반환하는 **형변환 메서드**
```java
Date d = Date.from(instant);
```

2. `of` : 여러 매개변수 받아 적합한 타입의 인스턴스를 반환하는 **집계 메서드**
```java
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
```

3. `valueOf` : `from` 과 `of` 의 더 자세한 버전
```java
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
```

4. `instance` / `getInstance` : 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스 보장하지 X
```java
StackWalker luke = StackWalker.getInstance(options);
```

5. `create` / `newInstance` : 매번 새로운 인스턴스를 생성해 반환함을 보장
```java
Object newArray = Array.newInstance(classObject, arrayLen);
```

6. `getType` : `getInstance` 와 같으나, 생성 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할때 사용
```java
FileStore fs = Files.getFileStore(path)
```

7. `newType` : `newInstance` 와 같으나, 생성 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할때 사용
```java
BufferedReader br = Files.newBufferedReader(path);
```

8. `type` : `getType` / `newType` 간결 버전
```java
List<Complaint> litany = Collections.list(legacyLitany);
```


참고자료
[정적 팩토리 메서드는 왜 사용할까?](https://tecoble.techcourse.co.kr/post/2020-05-26-static-factory-method/)