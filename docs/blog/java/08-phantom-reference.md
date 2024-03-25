## PhantomReference란?

`PhantomReference`는 `Java`의 `Reference` 클래스의 하위 클래스 중 하나로, `Java` 객체의 참조를 간접적으로 유지하면서 해당 객체가 가비지 컬렉션될 때 알림을 받을 수 있도록 해주는 클래스입니다.

`PhantomReference`를 생성할 때는 해당 객체의 참조와 함께 `ReferenceQueue`를 함께 전달해야 합니다. 이렇게 생성된 `PhantomReference`는 객체의 참조를 간접적으로 유지하지만, 실제 객체를 참조하는 것은 아닙니다. 그러므로 해당 객체는 가비지 컬렉션 대상이 됩니다.

`PhantomReference`가 알림을 받기 위해서는 해당 객체가 가비지 컬렉션될 때, `PhantomReference`가 참조하는 객체와 함께 `ReferenceQueue`에 등록되어야 합니다. 이 때, 알림을 받기 위해 `ReferenceQueue`에서 `PhantomReference`를 꺼내 처리하는 작업이 필요합니다.

`PhantomReference`는 주로 메모리 해제 작업이 필요한 경우에 사용됩니다. 예를 들어, 파일이나 네트워크 연결과 같은 리소스를 가지고 있는 객체를 생성한 후, 해당 객체가 가비지 컬렉션될 때 해당 리소스를 해제하는 작업을 수행하기 위해 `PhantomReference`를 사용할 수 있습니다. 이렇게 하면 객체가 가비지 컬렉션될 때 자동으로 해당 리소스를 해제할 수 있으므로, 메모리 누수와 같은 문제를 방지할 수 있습니다.

## PhantomReference를 이용해 가비지 컬렉션 될 때 알림 받는 방법

큰 메모리를 가진 Object를 생성한 후, `PhantomReference`를 이용하여 해당 객체가 가비지 컬렉션 될 때 알림을 받는 예제 코드는 아래와 같습니다.

```java
import java.lang.ref.PhantomReference;
import java.lang.ref.Reference;
import java.lang.ref.ReferenceQueue;
import java.time.LocalDateTime;

class PhantomReferenceExample {
    public static void main(String[] args) {
        System.out.println("startTime = " + LocalDateTime.now());
        // 큰 메모리를 가진 Object 생성
        LargeObject largeObject = new LargeObject();

        // ReferenceQueue 생성
        ReferenceQueue<LargeObject> referenceQueue = new ReferenceQueue<>();

        // PhantomReference 생성
        PhantomReference<LargeObject> phantomReference = new PhantomReference<>(largeObject, referenceQueue);

        // largeObject 참조 해제
        largeObject = null;

        // 가비지 컬렉션 수행
        System.gc();

        // ReferenceQueue에서 PhantomReference를 가져와서 처리
        Reference<? extends LargeObject> referenceFromQueue;
        while ((referenceFromQueue = referenceQueue.poll()) != null) {
            if (referenceFromQueue == phantomReference) {
                // 여기에서 해당 객체의 리소스를 해제하거나, 반납하는 작업을 수행
                System.out.println("LargeObject 객체가 가비지 컬렉션 되었습니다.");
                System.out.println("collectedTime = " + LocalDateTime.now());
            }
        }
    }
}

class LargeObject {
    private final byte[] data = new byte[1024 * 1024 * 100]; // 100MB 크기의 배열
}
```

위 코드에서 `PhantomReference`를 생성할 때, 생성한 `ReferenceQueue`를 전달하면 가비지 컬렉션이 발생할 때 `PhantomReference`가 이 `ReferenceQueue`에 등록됩니다. 그리고 가비지 컬렉션 이후에는 `ReferenceQueue`에서 `PhantomReference`를 가져와서 처리하면 됩니다. 이 때, 가져온 Reference가 생성한 `PhantomReference`와 같은지를 확인하여 해당 객체가 가비지 컬렉션되었는지를 판단할 수 있습니다.

실행 결과는 다음과 같습니다.

```text
startTime = 2023-05-11T00:02:56.568798
LargeObject 객체가 가비지 컬렉션 되었습니다.
collectedTime = 2023-05-11T00:02:56.608290
```

여러 차례 테스트 해봤는데 가비지 컬렉션 되지 않은 경우도 많이 있었습니다. Thread.sleep으로 충분한 시간을 주는 등 가비지 컬렉션이 발생할 때까지 기다리면 위와 같은 결과를 계속 확인할 수 있을 거 같네요.

배치 작업이나 bulk API 요청 등을 할 때 메모리를 고려하여 `chunkSize`를 설정하는데, 가비지 컬렉터가 어느 정도 용량까지 잘 커버하면서 수거해가는지 확인하는데 위 코드를 응용하면 좋을 거 같네요.