다형성을 사용할 때 자식 또는 구현체의 객체가 어떤 타입인지 알아보는 방법은 익히 알려져있습니다.

다음과 같은 두 개의 클래스가 있을 때,

```java
class Shape {
  // 생략
}

class Triangle extends Shape {
  // 생략
}
```

`Triangle` 객체가 어떤 타입인지 알아보려면 아래처럼 확인할 수 있습니다.

```java
class Scratch {

  public static void main(String[] args) {
    Triangle triangle = new Triangle();
    System.out.println(triangle.getClass());
    System.out.println(triangle instanceof Shape);
    System.out.println(triangle instanceof Triangle);
    System.out.println(triangle.getClass() == Triangle.class);
  }
}
```

이 코드를 실행시키면 결과는 아래와 같습니다.

```text
class Triangle
true
true
true
```

여기서 `instanceof`는 객체가 특정 타입인지 확인하는 연산인데 `triangle.getClass()`는 객체가 아닌 타입을 반환하므로 `Triangle.class` 라는 타입과 비교를 해야합니다.

```java
Class<? extends Triangle> triangleClass = triangle.getClass();
```

그렇다면 이 타입이 `Shape`의 subclass라는 것은 어떻게 알 수 있을까요?

```java
System.out.println(triangle.getClass() == Shape.class);
```

이렇게 타입으로 비교했을 때 결과는 어떻게 나올까요?

`==` 연산을 사용했으므로 너무 당연히 `false`가 나와야겠죠?

실제론 **"incomparable types: java.lang.Class<capture#1 of ? extends Triangle> and java.lang.Class<Shape>"** 이런 에러가 발생했습니다.

아무튼 타입간의 단순 `==` 연산으로는 부모-자식 관계라는 것을 확인할 수 없습니다.

이럴 때는 `Class<T>` 클래스에서 제공하는 메서드를 사용하면 됩니다.

```java
class Scratch {

  public static void main(String[] args) {
    Shape shape = new Shape();
    Triangle triangle = new Triangle();
    System.out.println(triangle.getClass().isAssignableFrom(Shape.class));
    System.out.println(Shape.class.isAssignableFrom(triangle.getClass()));
  }
}
```

일부러 두 가지 경우를 모두 테스트했는데 결과를 예측해보세요!

정답은 바로

```text
false
true
```

입니다.

`isAssignableFrom` 메서드는 파라미터로 전달될 타입이 메서드를 호출한 타입에 할당될 수 있다는 뜻이므로 헷갈리시면 안 됩니다.

**한 줄 요약**

```
부모타입.isAssignableFrom(자식타입);
```