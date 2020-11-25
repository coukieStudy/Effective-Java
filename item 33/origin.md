# 타입 안전 이종 컨테이너를 고려하라
- 컬렉션이나 단일 원소 컨테이너 같은 일반적인 제네릭의 쓰임에서 매개변수화되는 대상은 원소가 아닌 컨테이너 자기 자신이다.
    - 제네릭 타입: `Collection<E>`
    - 매개변수화 타입: `Collection<String>` (실체화된 제네릭 타입)
- 그러므로 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수는 제한된다.
    - `Collection<String>`에서는 `String` 타입만 사용할 수 있다.
- 만약 한 컨테이너에서 여러 종류의 타입을 사용하고 싶다면? 타입 안전 이종 컨테이너 패턴 (type safe heterogeneous container pattern)을 사용하라.
## 예시: `Favorites` 클래스
```java
// 타입별로 즐겨 찾는 인스턴스를 저장하고 검색하는 클래스
public class Favorites {
    public <T> void putFavorite(Class<T> type, T instance);
    public <T> getFavorite(Class<T> type);
    // type과 같은 class 리터럴을 타입 토큰이라 한다
}

public static void main(String[] args) {
    Favorites f = new Favorites();

    f.putFavorite(String.class, "Java"); // T = String
    f.putFavorite(Integer.class, 0xcafebabe); // T = Integer
    f.putFavorite(Class.class, Favorites.class); // T = Class

    String favoriteString = f.getFavorite(String.class);
    int favoriteInteger = f.getFavorite(Integer.class);
    Class<?> favoriteClass = f.getFavorite(Class.class);

    System.out.printf("%s %x %s%n", favoriteString, favoriteInteger, favoriteClass.getName());
    // Java cafebebe Favorites
}

// Favorites의 구현
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objectts.requireNonNull(type), instance);
    }

    public <T> getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```

- 맵의 키가 `Class<?>`라는 건 맵에 아무것도 넣을 수 없다는 뜻이 아니다. 왜냐? `?`가 아니라 `Class<?>`이기 때문. 오히려 아무거나 들어와도 된다는 뜻이다.
    - 타입이 `Class<?>`일 때 `Class<String>`이 들어와도 문제없다. 원래 타입이 `Class<?>`인 인스턴스에 변경을 가하는 것이 아니기 때문.
- 키(`Class<?>`)와 값(`Object`) 사이의 타입 관계는 보증되지 않는다. 하지만 `putFavorite`와 `getFavorite`를 통해 관계가 실질적으로 보증된다.
    - `putFavorite`에서 인스턴스 인자를 `T` 타입으로 받지만 맵에 넣을 때 `Object`로 넣기 때문에 타입 링크 정보가 버려진다.
    - `getFavorite`에서 `type.cast`를 통해 타입 링크를 되살린다. (사실 그냥 `(T)`로 형변환하는 거랑 똑같음)
    ```java
    public class Class<T> {
        // 제네릭을 이용한 동적 형변환
        @SuppressWarnings("unchecked")
        public T cast(Object obj) {
            if (obj != null && !isInstance(obj))
                throw new ClassCastException(cannotCastMsg(obj));
            return (T) obj;
        }
    }
    ```
### 제약 1: 타입 안정성이 깨질 가능성 존재
- 클라이언트가 로 타입으로 억지를 부리면 `Favorites`의 타입 안정성이 깨질 수 있다.
```java
f.putFavorite((Class)Integer.class, "Not Integer instance");
int favoriteInteger = f.getFavorite(Integer.class);
```
- 이런 문제는 `HashMap`이나 `HashSet`같이 자바에서 제공하는 컬렉션에도 똑같이 존재하니 그냥 넘어가도 될 듯?
- 못 넘어가겠으면 동적 형변환을 해주면 된다.
```java
public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), type.cast(instance));
}
```
- 자바에서 제공하는 `checkedMap`이나 `checkedSet`이 똑같은 일을 하는 컬렉션을 제공한다.
```java
public class Collections {
    static class CheckedCollection<E> implements Collection<E>, Serializable {
        final Collection<E> c;
        final Class<E> type;

        @SuppressWarnings("unchecked")
        E typeCheck(Object o) {
            if (o != null && !type.isInstance(o))
                throw new ClassCastException(badElementMsg(o));
            return (E) o;
        }

        public boolean add(E e) { return c.add(typeCheck(e)); }
    }

    static class CheckedSet<E> extends CheckedCollection<E> implements Set<E>, Serializable {}

    public static <E> Set<E> checkedSet(Set<E> s, Class<E> type) {
        return new CheckedSet<>(s, type);
    }
}
```

### 제약 2: 실체화 불가 타입에 사용 불가
- 즉 런타임에 타입이 소거되는 제네릭은 `Favorites`에 저장할 수 없다.
- `List<String>`과 `List<Integer>`는 `List.class`라는 같은 타입 토큰을 공유하기 때문이다.
- 해결책?: 슈퍼 타입 토큰
    - `Class` 클래스의 `getGenericSuperClass()`를 이용한 우회로
    - https://gafter.blogspot.com/2006/12/super-type-tokens.html
    - https://homoefficio.github.io/2016/11/30/클래스-리터럴-타입-토큰-수퍼-타입-토큰/
- 슈퍼 타입 토큰의 한계: static 타입만 반영할 수 있다.
    - http://gafter.blogspot.com/2007/05/limitation-of-super-type-tokens.html
    - https://stackoverflow.com/questions/21987295/using-spring-resttemplate-in-generic-method-with-generic-parameter

## 한정적 타입 토큰
- `Favorites` 같은 클래스가 받는 타입을 제한하고 싶다면? 한정적 타입 토큰을 사용하면 된다.
- 예시: `AnnotatedElement`의 `getAnnotation` 메서드
```java
public interface AnnotatedElement {
    // 클래스나 메서드, 필드에 달려 있는 애너테이션을 반환하는 메서드
    public <T extends Annotation> T getAnnotation(Class<T> annotationType);
}
```
- 만약 `Class<?>` 타입의 객체를 `getAnnotation`에 넘기고 싶다면? `Class<? extends Annotation>`으로 형변환하면 되는데, 직접 하면 경고가 뜬다. `Class` 객체의 `asSubclass` 메서드를 사용하면 경고 없이 컴파일된다.
```java
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
    Class<?> annotationType = null;
    try {
        annotationType = Class.forName(annotationTypeName);
    } catch (Exception ex) {
        throw new IllegalArgumentException(ex);
    }
    return element.getAnnotation(
        annotationType.asSubclass(Annotation.class)
    );
}