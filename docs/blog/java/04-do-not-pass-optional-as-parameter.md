자바 프로그래밍을 하다보면 `Optional`을 파라미터로 전달했을 때 컴파일러가 경고(노란줄)를 표시합니다. (IntelliJ에서는 표시해주는데 다른 IDE는 어떤지 잘 모르겠네요)

경고를 확인하기 위해 간단한 코드를 작성해보면,

```java
  public void foo(String nullable) {
    bar(Optional.of(nullable));
  }

  private void bar(Optional<String> s) {
    // do something
  }
```

```text
'Optional<String>' used as type for parameter 's' 
```

바로 이런 내용을 확인할 수 있습니다.

그렇다면 왜 `Optional`을 파라미터로 전달하면 안 되는 것일까요?

결론부터 말씀드리면 장점보다 단점이 많기 때문입니다.

먼저 장점으로는 전달할 당시에 별 생각 없이 전달할 수 있다는 점이 있습니다. 어차피 전달할 메서드에서 처리할 것이기 때문에 메서드에 해당 변수나 값에 대한 처리를 위임할 수 있습니다.

반대로 메서드 내에서 조건절 작성을 강제하도록 유도하는 것은 비생산적입니다. 또한 컴파일러 입장에서 `Optional`로 감싸는 것은 불필요한 `wrapping`을 유발하는 경우가 생길 수 있습니다. 게다가 `nullable`한 파라미터와 비교하여 `Optional`은 처리하는데 비용이 더 많이 듭니다. 마지막으로 필수 파라미터를 `null`로 전달할 가능성이 높아져 실수를 유발하기 쉽습니다.

애초에 단순하게 생각하면 `Optional`을 파라미터로 전달한다는 것 자체가 세 가지 상태를 가질 수 있음을 나타냅니다. (1) `Optional` 자체가 `null`인 경우, (2) `Optional.isPresent`가 `true`인 경우(실제 값을 가지는 경우)와 (3) `false`인 경우(`Optional.empty()`) 이렇게 총 세 가지 입니다.

```java
  public void foo(String nullable) {
    bar(null); // (1)
    bar(Optional.ofNullable(nullable)); // (2)
    bar(Optional.of("nonnull")); // (3)
  }

  private void bar(Optional<String> s) {
    if (s != null) { // Optional을 null과 비교해야 함, 역시 컴파일러가 경고로 알려줌
      
    }
  }
```

(1)의 경우 바로 위에 작성한 코드처럼 `Optional`을 `null`과 비교하는 로직이 추가되어야 합니다. `null`을 전달하는 경우를 방지하기위해 `@NotNull` 애너테이션 등을 사용한다면 `NotNull`이지만 `Nullable`한 아이러니한 상황이 연출됩니다.

```java
  private static void test(@NotNull Optional<String> s) {
    if(!s.isPresent()) { // NotNull이지만 s가 가진 value 자체는 null이 될 수 있음
      
    }    
  }
```

따라서 `Optional`을 파라미터로 전달하는 대신 필요한 경우 전달한 파라미터를 메서드 내부에서 `Optional`로 `wrapping` 해주어야 합니다.

사실 의도적으로 `Optional`을 전달하는 경우는 흔치 않을 거 같은데, 코드를 작성하다가 특정 부분을 메서드로 추출하는 단축키를 눌렀다가 자동으로 `Optional` 파라미터를 전달하게 되는 경우가 있는데, 이 메서드를 다른 누군가가 재활용한다면 의도에 맞게 사용되지 않을 수 있으므로 꼭 주의하셔야 합니다. 