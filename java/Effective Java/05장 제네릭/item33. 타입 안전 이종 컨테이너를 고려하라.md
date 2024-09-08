# item33. 타입 안전 이종 컨테이너를 고려하라

### 타입 안정 이종 컨테이너란 무엇일까?
타입 안전 이종 컨테이너 말이 참 어렵습니다.</br>
우선 컨테이너를 알아보도록 하죠.</br>
컨테이너는 다른 객체를 넣을 수 있는 또 다른 객체를 의미합니다.</br>
`Optional`, `Map`, `Set`이 이에 속합니다.

우리가 지금까지 제네릭은 오로지 한가지 타입만 넣을 수 있는 컨테이너를 만들었습니다.</br>

```java
public class Favorites<T> {
    List<T> values;

	public static void main(String[] args) {
        Favorites<String> names = new Favorites<>();
		names.values.add("Java");
		names.values.add(2123); // 컴파일 에러
		
        Favorites<Integer> numbers = new Favorites<>();
	}
}
```

경우에 따라서는 이종 컨테이너(같은 종이 아닌 것(동종 <=> 이종))를 만들어야 할 때가 있습니다.</br>

```java
public class Favorites {
	private Map<Class, Object> map = new HashMap<>();
	
	public void put(Class type, Object value) {
		this.map.put(type, value);
    }
	
	public Object get(Class type) {
        return this.map.get(type);
    }

	public static void main(String[] args) {
		Favorites favorites = new Favorites();
		
		// String 타입에 문자열만 넣고 싶은데 아래의 코드가 가능하다.
		favorites.put(String.class, 2);
	}
}
```
이종 컨테이너를 만들려면 위에 코드처럼 작성하면 됩니다</br>
하지만 위의 코드는 타입 safety가 보장되지 않는다는 문제가 있습니다.</br>

그렇다면 type safety를 어떻게 보장할 수 있을까요?</br>
```java
public class Favorites {
	private Map<Class<?>, Object> map = new HashMap<>();
	
	public <T> void put(Class<T> type, T value) {
		this.map.put(type, value);
    }
	
	public <T> T get(Class<T> type) {
        return type.cast(this.map.get(type));
    }

	public static void main(String[] args) {
		Favorites favorites = new Favorites();
        favorites.put(String.class, "Java");
        favorites.put(Integer.class, 2);
		
		String str = favorites.get(String.class);
		Integer num = favorites.get(Integer.class);
	}
}
```
위의 코드처럼 작성하면 type safety를 보장할 수 있습니다.</br>
이것을 타입 안전 이종 컨테이너라고 합니다.</br>

위에 코드에서 조금 더 개선하자면</br>
put 할 때 requireNonNull을 사용하면 좋습니다.</br>
```java
public <T> void put(Class<T> type, T value) {
    this.map.put(Objects.requireNonNull(type), value);
}
```

하지만 타입 안전 이종 컨테이너를 구현한 Favorite 클래스에서</br>
타입 안전을 의도적으로 깨트릴 수 있는 단점이 있습니다.

```java
public static void main(String[] args) {
    Favorites favorites = new Favorites();
	
    favorites.put((Class)String.class, 1);
    
    String str = favorites.get(String.class);
}
```

위의 코드처럼 로타입을 써서 타입 안전을 깨트릴 수 있습니다.</br>
type을 지정하지 않고 Class 타입으로 넘기게 된다면, type erasure(타입 이레이저)로 인해</br>
다 Object 타입으로 변환되기 때문에 아무 값이나 넣을 수 있게 됩니다.</br>

이처럼 하게되면 컴파일 경고가 발생하지 않고 다른 타입의 값을 넣을 수 있게 되며</br>
값을 꺼낼 때(favorites.get(String.class)) ClassCastException이 발생하게 됩니다.

이러한 문제를 막고싶다면 값을 넣을때 한번 더 체크해주는 방법이 있습니다.</br>
```java
public <T> void put(Class<T> type, T value) {
    this.map.put(Objects.requireNonNull(type), type.cast(value));
}
```
이렇게 되면 꺼낼때가 아니라, 값을 넣을때 Exception이 발생하기 때문에 조금 더 빨리 알아차릴 수 있다,</br>
그러나 이러한 방법도 근본적으로 컴파일 에러로 나타낼 수 있는 방법은 존재하지 않습니다.

타입 안정 이종 컨테이너를 사용했을때의 또 다른 단점은,</br>
List를 사용할 때 입니다.

```java
public class Favorites {
	private Map<Class<?>, Object> map = new HashMap<>();

	public <T> void put(Class<T> type, T value) {
		this.map.put(Objects.requireNonNull(type), type.cast(value));
	}
	
	public <T> T get(Class<T> type) {
        return type.cast(this.map.get(type));
    }

	public static void main(String[] args) {
		Favorites favorites = new Favorites();
		favorites.put(List.class, List.of(1, 2, 3));
		favorites.put(List.class, List.of("a", "b", "c"));
		
		List list = favorites.get(List.class);
		list.forEach(System.out::println);
	}
}
```
위 코드처럼 키 값을 중복으로 넣게 되면 덮어씌워지게 됩니다.</br>
하지만, type을 `List<String>.class`, `List<Integer>.class`와 같이 구분하고 싶어도</br>
문법이 허용하지 않습니다.




#
#
#
# 슈퍼타입토큰(Super Type Token)


이 글은 토비 슈퍼토큰 동영상(https://www.youtube.com/watch?v=01sdXvZSjcI)을 보고 정리했다.

## 제네릭
```java
public class GenericTest {
    static class Generic<T> { // T를 타입 파라미터라고 부름
        T value;
        void set (T t) {}
    }

    public static void main(String[] args) {
        Generic<String> s = new Generic<>();
        s.value = "String";
        s.set("String");

        Generic<Integer> i = new Generic<>();
        i.value = 10;
        i.set(10);
    }
}
```
제네릭으로 선언된 클래스 같은것을 선언하는것을 parameterized type 이라고 얘기한다.

```java
public class GenericTest {

    static <T> T create(Class<T> clazz) throws Exception {
        return clazz.newInstance();
    }

    public static void main(String[] args) throws Exception {
        Object o1 = create(String.class);
        System.out.println(o1.getClass());
    }
}
```
## 타입토큰(Type Token)
```java
    static class TypeUnSafeMap {
        Map<String, Object> map = new HashMap<>();
        void run() {
            map.put("a", "a");
            map.put("b", 1);

            String s = (String)map.get("a");
            Integer i = (Integer) map.get("b");
        }
    }
```
위와 같은 코드는 위험하다. 실행 중에 예상치 못했던 타입 에러가 날 수 있다.

```java
public class TypeToken {

    static class TypeSafeMap {
        Map<Class<?>, Object> map = new HashMap<>();

        <T> void put (Class<T> clazz, T value) {
            map.put(clazz, value);
        }

        <T> T get(Class<T> clazz) {
            return clazz.cast(map.get(clazz));
        }
    }

    public static void main(String[] args) throws Exception {
        TypeSafeMap m = new TypeSafeMap();
        m.put(String.class, "String");
        m.put(Integer.class, 1);
        m.put(List.class, Arrays.asList(1,2,3));

        System.out.println(m.get(String.class));
        System.out.println(m.get(Integer.class));
        System.out.println(m.get(List.class));
    }
}
```
반면 위의 TypeSafeMap 클래스는 강제로 형변환을 하는게 하나도 없기 때문에 안전하다. 그리고 특정 타입의 클래스 정보를 넘겨서 타입 안전성을 꿰하도록 코드를 작성하는 기법을 TypeToken이라고 한다.

하지만 위의 코드는 한계가 있다(같은 List 타입이라 덮어씌워지게 된다).

```java
m.put(List.class, Arrays.asList(1,2,3));
m.put(List.class, Arrays.asList("a", "b", "c")); // 덮어씌움
```
그래서 아래와 같은 방식으로 하면 될 거 같지만 컴파일 에러가 발생한다.

```java
m.put(List<Integer>.class, Arrays.asList(1,2,3));
m.put(List<String>.class, Arrays.asList("a", "b", "c"));
```

클래스 리터럴(.class)로 List<Integer> 에 대한 클래스 오브젝트를 가져올 때 타입 파라미터 적용한거를 구분해서 가져올 수 없다. Class 타입에는 제네릭 타입 파라미터에 대한 정보가 없다. 이 문제는 스프링 restTemplate에서 Http response body 정보를 자바 객체로 컨버팅 할 때도 발생한다.

## 슈퍼타입토큰(Super Type Token)
Neal Gafter가 만든 기법이다.

```java
public class SuperTypeToken {

    static class Sup<T> {
        T value;
    }

    public static void main(String[] args) throws Exception {
        Sup<String> s = new Sup<>();
        System.out.println(s.getClass().getDeclaredField("value").getType()); // class java.lang.Object
    }
}
```
리플렉션을 통해서도 타입 파라미터 정보를 얻어올 수 없다. eraser에 의해서 타입 파라미터 정보가 런타임시 사라져 버린다.

```java
public class SuperTypeToken {

    static class Sup<T> {
        T value;
    }

    static class Sub extends Sup<Map<List<?>, Set<String>>> {
    }

    public static void main(String[] args) throws Exception {
        Sub b = new Sub();
        Type t = b.getClass().getGenericSuperclass();
        ParameterizedType pType = (ParameterizedType)t;
        System.out.println(pType.getActualTypeArguments()[0]); // java.util.Map<java.util.List<?>, java.util.Set<java.lang.String>>
    }
}
```
하지만 위와 같은 방식을 사용하면 타입 파라미터 정보를 가져올 수 있다.

비교를 해보면 Sup<String> s = new Sup<>();는 클래스에 인스턴스를 만들면서 타입을 준거다. 이렇게 작성한 코드는 하위호환성 문제(제네릭이 없는 1.5 이하 버전) 때문에 타입 정보를 런타임에 삭제해버린다. static class Sub extends Sup<String> {} 이 방식은 새로운 타입을 정의하면서 슈퍼클래스를 제네릭 클래스로 하고 타입 파라미터를 지정했다. 그러면 리플렉션을 통해서 런타임에 접근할 수 있도록 바이트코드에 남아있다.

```java
public class SuperTypeToken {

    static class Sup<T> {
        T value;
    }

    public static void main(String[] args) throws Exception {
        // 로컬 클래스
        // class Sub extends Sup<Map<List<?>, Set<String>>> {}

        // 익명 클래스
        // new Sup<Map<List<?>, Set<String>>>() {};

        // 위의 방식처럼 익명 클래스를 바로 이용하면 된다
        Sup b = new Sup<Map<List<?>, Set<String>>>() {};
        Type t = b.getClass().getGenericSuperclass();
        ParameterizedType pType = (ParameterizedType)t;
        System.out.println(pType.getActualTypeArguments()[0]); // java.util.Map<java.util.List<?>, java.util.Set<java.lang.String>>
    }
}
```
이를 이용해 로컬 클래스로 만들수도 있고 익명 클래스로 만들수도 있다. 이를 이용해서 타입 파라미터 정보를 얻어오는 클래스를 만들어보자.

```java
public class SuperTypeToken {

    static class TypeReference<T> {
        Type type;

        public TypeReference() {
            Type sType = getClass().getGenericSuperclass();
            if (sType instanceof ParameterizedType) {
                this.type = ((ParameterizedType)sType).getActualTypeArguments()[0];
            } else {
                throw new RuntimeException();
            }
        }
    }

    public static void main(String[] args) throws Exception {

        // ParameterizedType이 아니다. 런타임에 타입 정보가 안남아있기 때문에 타입 정보를 Object로 인식한다. 그래서 RuntimeException이 발생한다.
        // TypeReference t = new TypeReference<List<String>>();

        // TypeReference를 상속받은 익명 클래스를 이용하기 때문에 바디부분{}이 필요하다.
        TypeReference t = new TypeReference<List<String>>(){};
        System.out.println(t.type); // java.util.List<java.lang.String>
    }
}
```
이제 저 아이디어를 이용해서 기존 TypeToken이 가졌던 한계(타입 파라미터 정보를 가져올 수 없었던)를 해결해보자.

```java
public class SuperTypeToken {

    static class TypeSafeMap {
        Map<TypeReference<?>, Object> map = new HashMap<>();

        <T> void put (TypeReference<T> tr, T value) {
            map.put(tr, value);
        }

        <T> T get(TypeReference<T> tr) {
            if (tr.type instanceof  Class<?>)
                return ((Class<T>)tr.type).cast(map.get(tr));
            else
                return ((Class<T>)((ParameterizedType)tr.type).getRawType()).cast(map.get(tr));
        }
    }

    static class TypeReference<T> {
        Type type;

        public TypeReference() {
            Type sType = getClass().getGenericSuperclass();
            if (sType instanceof ParameterizedType) {
                this.type = ((ParameterizedType)sType).getActualTypeArguments()[0];
            } else {
                throw new RuntimeException();
            }
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass().getSuperclass() != o.getClass().getSuperclass()) return false;
            TypeReference<?> that = (TypeReference<?>)o;
            return type.equals(that.type);
        }

        @Override
        public int hashCode() {
            return Objects.hash(type);
        }
    }

    public static void main(String[] args) throws Exception {
        TypeSafeMap m = new TypeSafeMap();
        m.put(new TypeReference<String>(){}, "String");
        m.put(new TypeReference<Integer>(){}, 1);

        m.put(new TypeReference<List<Integer>>(){}, Arrays.asList(1,2,3));
        m.put(new TypeReference<List<String>>(){}, Arrays.asList("a", "b", "c"));

        System.out.println(m.get(new TypeReference<String>(){}));
        System.out.println(m.get(new TypeReference<Integer>(){}));
        System.out.println(m.get(new TypeReference<List<Integer>>(){}));
        System.out.println(m.get(new TypeReference<List<String>>(){}));
    }
}
```
지금까지는 슈퍼타입토큰에 대한 원리를 설명했고 스프링이 제공하는 ParameterizedTypeReference를 이용해서 쉽게 사용할 수 있다.

```java
    import org.springframework.core.ParameterizedTypeReference;

    public static void main(String[] args) throws Exception {
        ParameterizedTypeReference<?> typeRef = new ParameterizedTypeReference<List<Map<Set<Integer>, String>>>() {};
        System.out.println(typeRef.getType()); // java.util.List<java.util.Map<java.util.Set<java.lang.Integer>, java.lang.String>>
    }
```
그리고 restTemplate에서도 response body에 있는 정보를 객체로 컨버팅할 때 이용된다.

```java
    List<String> response = restTemplate.exchange(url, HttpMethod.POST, httpEntity,
        new ParameterizedTypeReference<List<String>>(){}).getBody();
```
추가 방송을 통한 개선 및 스프링 ResolvableType

TypeSafeMap에서 key로 TypeReference 클래스를 뒀기 때문에 equals와 hashCode를 재정의 해줬어야 됐는데, key값을 TypeReference의 type으로 두면 재정의가 필요 없어진다.

```java
public class SuperTypeToken {

    static class TypeSafeMap {
        Map<Type, Object> map = new HashMap<>();

        <T> void put (TypeReference<T> tr, T value) {
            map.put(tr.type, value);
        }

        <T> T get(TypeReference<T> tr) {
            if (tr.type instanceof  Class<?>)
                return ((Class<T>)tr.type).cast(map.get(tr.type));
            else
                return ((Class<T>)((ParameterizedType)tr.type).getRawType()).cast(map.get(tr.type));
        }
    }

    static class TypeReference<T> {
        Type type;

        public TypeReference() {
            Type sType = getClass().getGenericSuperclass();
            if (sType instanceof ParameterizedType) {
                this.type = ((ParameterizedType)sType).getActualTypeArguments()[0];
            } else {
                throw new RuntimeException();
            }
        }
    }

    public static void main(String[] args) throws Exception {
        TypeSafeMap m = new TypeSafeMap();
        m.put(new TypeReference<String>(){}, "String");
        m.put(new TypeReference<Integer>(){}, 1);

        m.put(new TypeReference<List<Integer>>(){}, Arrays.asList(1,2,3));
        m.put(new TypeReference<List<String>>(){}, Arrays.asList("a", "b", "c"));

        System.out.println(m.get(new TypeReference<String>(){}));
        System.out.println(m.get(new TypeReference<Integer>(){}));
        System.out.println(m.get(new TypeReference<List<Integer>>(){}));
        System.out.println(m.get(new TypeReference<List<String>>(){}));
    }
}
```
TypeReference의 Type은 JVM이 고유하게 만들어서 관리한다. 제네릭 정보가 들어가 있으니까 클래스처럼 처음부터 클래스 로딩할 때 인스턴스를 만들어서 가지고 있지는 않는다. 그런데 새로운 제네릭 타입을 TypeReference에 넘겨서 그 정보를 찾으려고 시도를 하면 그때 이 인스턴스가 없으면 새로운 걸 하나 만들어 낸다고 한다. 따라서 key값으로 줘도 된다.

스프링 4.0부터 추가된 ResolvableType을 이용하면 다양한 타입에 대한 접근이 편리하다.

```java
ResolvableType rt = ResolvableType.forInstance(new TypeReference<List<Map<Set<Integer>, String>>>(){});
System.out.println(rt.getSuperType().getGenerics().length); // 1
System.out.println(rt.getSuperType().getNested(1)); // com.common.util.SuperTypeToken$TypeReference<java.util.List<java.util.Map<java.util.Set<java.lang.Integer>, java.lang.String>>>
System.out.println(rt.getSuperType().getNested(2)); // java.util.List<java.util.Map<java.util.Set<java.lang.Integer>, java.lang.String>>
System.out.println(rt.getSuperType().getNested(3)); // java.util.Map<java.util.Set<java.lang.
```