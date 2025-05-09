### 불변 클래스
- 인스턴스 내부 값을 수정할 수 없는 클래스
- 파괴될때까지 절대 달라지지 않아서 안전함

### 불변 클래스를 만들 위해서
- 상태 변경 메서드 제공 X
- 클래스 확장할 수 없도록 함
	- final class로 만들기
	- 모든 생성자를 private 혹은 package-private로 만들고, public 정적 팩터리를 제공
```java
 public class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
    public static Complex valueOf(double re, double im) {
        return new Complex(re,im);
    }
}
```
-> 이 불변 객체는 사실상 final,  public 이나 protected 생성자가 없으니 다른 패키지에서는 이 클래스를 확장하는 게 불가능

- 모든 필드 final로 선언
- 모든 필드 private로 선언
- 자신외에 내부 가변 컴포넌트에 접근 불가능하도록 함


### 불변 클래스 장점
- 가변 클래스보다 설계하고 구현하고 사용하기 쉬움
- 스레드 안전하기 때문에 동기화 비용X
- 재활용과 캐시를 통해서 성능 향상이 가능함
- 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면, 불변식을 유지하기 쉽다 - 불변 객체는 맵Map의 키와 집합Set의 원소로 쓰기에 적합
- 실패원자성을 제공 -> 상태가 변하지 않는 특성


### 불변 클래스 단점
- 변하지 않는 것이 단점 -> 값이 다르면 다 독립 객체로 존재해야 함