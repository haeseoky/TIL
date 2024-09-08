# Item. 25 톱레벨 클래스는 한 파일에 하나만 담으라

> 소스 파일 하나에는 반드시 톱레벨 클래스(혹은 톱레벨 인터페이스)를 하나만 담자
> 이 규칙만 따른다면 컴파일러가 한 클래스에 대한 정의를 여러 개 만들어 내는 일은 사라진다.
> 소스 파일을 어떤 순서로 컴파일하든 바이너리 파일이나 프로그램의 동작이 사라지지 않는다.

---

### 톱레벨 클래스란?

-   소스 파일의 가장 바깥에 선언된 클래스를 말한다.
-   중첩 클래스가 아닌 클래스를 말한다.

<br/>

### 예시 (따라 하지 말 것!)

```java
# Song.java
class Genre {
  static final String NAME = "POP";
}

class Song {
  static final String NAME = "SONG";
}
```

```java
# Genre.java
Genre.java
class Genre {
  static final String NAME = "K-POP";
}

class Song {
  static final String NAME = "SONG";
}
```

```java
# main.java
# 컴파일 에러 발생
public class Test {
  public static void main(String[] args) {
  System.out.println(Genre.NAME + Song.NAME);
  }
}
```

<br/>

### 해결 방법

-   여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스를 사용하는 방법이 있다.

```java
public class AB {
    private static class A {

    }

    private static class B {

    }
}
```

<br/>

---

### Appendix

**컴파일러의 동작:**

-   소스 코드를 전체적으로 분석하여 중간 코드(Intermediate code) 또는 목적 코드(Object code)를 생성하는 과정이다.
-   전체 소스 코드를 한 번에 번역하므로 실행 속도가 빠르다.
-   번역된 결과물은 CPU가 직접 실행할 수 있는 기계어로 변환된다.

**인터프리터의 동작:**

-   소스 코드를 한 줄씩 읽어들이고, 해당 코드를 즉시 실행하는 과정이다.
-   소스 코드를 한 줄씩 해석하고 실행하기 때문에 번역과 실행이 번갈아가면서 이루어진다.
-   실행 속도가 상대적으로 느리지만, 개발 과정에서 수정 및 디버깅이 용이하다.

**자바의 빌드 과정:**

-   자바의 빌드 과정은 컴파일과 실행의 과정이 섞여 있음
-   컴파일 단계에서 소스 코드를 중간 형태인 바이트 코드로 변환하고, 실행 단계에서 JVM을 통해 해당 바이트 코드를 인터프리터로 - 해석하여 실행하는 방식
-   이러한 방식으로 자바 언어는 운영체제의 종류와 상관없이 동일한 바이트 코드를 실행하여 플랫폼 호환성을 제공