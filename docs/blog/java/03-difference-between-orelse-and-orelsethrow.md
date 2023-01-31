`Optional`을 사용하다보면 마지막에 `orElse()` 또는 `orElseGet()`를 이용해 원래 값을 얻습니다.

그동안 저는 두 메서드의 차이가 단순히 전달해야하는 파라미터의 차이라고 생각했었습니다.

예를 들면 `orElse(defaultValue)`, `orElseGet(this::getDefaultValue)` (또는 `orElseGet(() -> getDefaultValue())`) 이런식으로 하나는 값을 전달하고 하나는 구현체를 전달하기 때문에 그냥 적절하게 사용하면 되겠다는 정도로만 생각했었습니다.

그러던 중 테스트 클래스를 작성하다가 `mocking`을 사용하였는데 정상적으로 동작하지 않았습니다.

<details>
<summary>본문과 크게 관련 없는 내용(어떤 테스트가 동작하지 않았는지)이라 접어두겠습니다.</summary>

테스트 할 클래스는 `Service` 레이어였기 떄문에 `mocking`을 사용했고, 플로우와 커버리지 정도만 확인하는 테스트였습니다.

`mocking`을 통해 특정 값을 반환하지 않게하여 `orElse`가 실행될 것으로 예상되는 코드와 반환하여 `orElse`가 실행되지 않을 것이라고 예상되는 코드 두 가지를 작성하였는데, 반환하도록 하였을 때 문제가 생겼습니다.

```java
@UseCase
@RequiredArgsConstructor
public class FooService {
  
  private final LoadFooPort loadFooPort;
  private final LoadBarPort loadBarPort;
  // 생략

  @Override
  public SpendingReportResponse getSpendingReport(Long userId) {
    FooEntity spendingReportEntity = loadFooPort.findByUserId(userId).orElse(loadFromBarPortThenSave(userId));
    // 생략
  }

  private FooEntity loadFromBarPortThenSave(Long userId) {
    BarResponse barResponse = loadBarPort.loadBar(userId);
    // 생략
  }

}
```

`service`에서 `outgoing persistence adapter`를 호출하여 `entity`를 얻고, 존재하지 않으면 다른 `outgoing port`를 통해 `entity`를 만들어 반환하는 메서드를 호출하게 작성하였기 때문에 존재할 경우 `loadBarPort`의 메서드가 호출되지 않았어야했지만 실제로는 호출되고 있었습니다.

</details>

직접 간단하게 테스트를 작성해서도 확인할 수 있습니다.

```java
import java.util.Optional;

class Scratch {

  public static void main(String[] args) {
    String origin = Optional.of("이건 출력 되어야 함").orElse(print());
    System.out.println("origin = " + origin);
  }

  private static String print() {
    System.out.println("이건 출력되지 않아야함");
    return "이건 출력되지 않아야함";
  }
}
```

이렇게 작성 후 애플리케이션을 실행시키면

```text
이건 출력되지 않아야함
origin = 이건 출력 되어야 함
```

`origin`의 값이 바뀌지 않았음에도 이렇게 둘 다 출력되는 것을 확인할 수 있습니다.

그 말은 곧 `Optional`이 가지고있는 값이 `null`이 아니더라도 `orElse` 내부에서 호출한 메서드가 실행되었다는 뜻이겠죠?

반면 `orElseGet`을 이용하면,

```java
import java.util.Optional;

class Scratch {

  public static void main(String[] args) {
    String origin = Optional.of("이건 출력 되어야 함").orElseGet(Scratch::print);
    System.out.println("origin = " + origin);
  }

  private static String print() {
    System.out.println("이건 출력되지 않아야함");
    return "이건 출력되지 않아야함";
  }
}
```

```text
origin = 이건 출력 되어야 함
```

원하는대로 `orElseGet`에 전달한 메서드가 호출되지 않는 것을 확인할 수 있습니다.

따라서 대부분의 상황에서 `orElseGet`을 이용하는 게 낫습니다.

제가 위에 collapse 처리한 글 내용 처럼 `orElse`를 이용해서 특정 메서드를 호출할 경우, 특히 DB(insert, update 등)나 외부 연동이 포함되어 있는 경우엔 더 큰 문제를 일으킬 수 있으므로 `orElseGet`을 사용하는 게 더 안전한고, 그만큼 호출을 줄여 성능적으로도 유리할 수 있습니다. (lazy loading과 유사)

반면 `orElse`의 경우 이미 만들어진 객체나 `primitive`한 값을 반환할 때 사용하면 콜스택을 줄일 수 있습니다.

---

정리하면, `Optional`의 `value`가 `null`인 경우 `orElse`, `orElseGet`의 동작은 동일하고, `null`이 아닌 경우 동일하게 동작하지 않습니다.

상황에 따라 `orElse`, `orElseGet`을 잘 사용할 필요가 있습니다.

