## Any

`Any` 메시지 타입은 `.proto`로 정의되지 않은 타입을 포함할 수 있게 합니다.

Any에는 바이트로 직렬화된 임의의 메시지와 URL을 포함하는데, 해당 URL은 메시지 타입에 대해 고유한 식별자 역할을 합니다.

Any 타입을 사용하기 위해선 `google/protobuf/any.proto`를 `import` 해야 합니다.

```protobuf
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

주어진 메시지 타입에 대한 기본 타입 URL은 type.googleapis.com/`packagename`.`messagename` 으로 구성됩니다.

다른 언어로 구현될 때는 런타임 라이브러리 헬퍼를 지원하여 해당 언어로 pack/unpack 하는 기능을 `type-safe`한 방법으로 지원합니다.

예를 들어, `Java`에서 `Any` 타입은 `pack()`, `unpack()` 접근자가 있고, `C++`에서는 `PackFrom()`, `UnpackTo()` 라는 메서드를 지원합니다.

```
// Storing an arbitrary message type in Any.
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// Reading an arbitrary message from Any.
ErrorStatus status = ...;
for (const Any& detail : status.details()) {
  if (detail.Is<NetworkErrorDetails>()) {
    NetworkErrorDetails network_error;
    detail.UnpackTo(&network_error);
    ... processing network_error ...
  }
}
```

만약에 `proto2` 문법을 사용중이라면 `Any` 타입은 `proto3` 메시지 타입을 포함할 수 있습니다.

## oneof

많은 필드가 있고 동시에 최대 하나의 필드가 설정되는 메시지가 있는 경우 `oneof`를 사용하여 적용하고 메모리를 절약할 수 있습니다.

`oneof` 필드는 `oneof` 공유 메모리의 모든 필드를 제외하고 일반 필드와 같으며 최대 한 필드를 동시에 설정할 수 있습니다.

`oneof`의 `member`를 설정하면 다른 모든 `member`는 자동으로 지워집니다.

선택한 언어에 따라 특별한 `case()` 또는 `WhatOneof()` 메서드를 사용하여 `oneof`애 어떤 값이 세팅되는지 확인할 수 있습니다.

### oneof 사용하기

`.proto`에서 `oneof`를 정의하려면 `oneof` 키워드 다음에 `oneof` 이름을 지정합니다.

```
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

그런 다음 `oneof` 정의에 필드를 추가합니다. `map` 필드 및 `repeated` 필드를 제외한 모든 타입의 필드를 추가할 수 있습니다.

생성된 코드에서 `oneof` 필드에는 일반 필드와 동일한 `getter` 및 `setter`가 있습니다.

그리고 어떤 값이 `oneof`에 세팅되었는지 확인할 수 있는 메서드를 제공합니다.

언어별로 어떻게 코드가 생성되는지는 [여기](https://developers.google.com/protocol-buffers/docs/reference/overview)를 참고하시면 됩니다.

### oneof 기능

* `oneof` 필드를 설정하면 다른 `oneof`의 모든 멤버가 자동으로 지워집니다.
* `oneof` 필드를 여러 개 설정하면 마지막으로 설정한 필드만 값이 유지됩니다.
```protobuf
SampleMessage message;
message.set_name("name");
CHECK(message.has_name());
message.mutable_sub_message();   // Will clear name field.
CHECK(!message.has_name());
```
* 만약에 파서가 같은 `oneof`의 멤버들을 발견한다면 마지막에 발견한 멤버만 파싱된 메시지에서 사용됩니다.
* `oneof`는 `repeated` 타입이 될 수 없습니다.
* `Reflection API`는 `oneof` 필드에도 동작합니다.
* `oneof` 필드를 `default value`로 설정하면 해당 `oneof` 필드의 "케이스"가 설정되고 값이 직렬화됩니다.

### 하위 호환성 이슈

필드 중 하나를 추가하거나 제거할 때 주의해야합니다.

`oneof`의 값을 검사할 때 `None` 또는 `NOT_SET`이 반환되면 `oneof`가 설정되지 않았거나 `oneof`의 다른 버전에 있는 필드로 설정되었음을 의미할 수 있습니다.

필드가 `oneof`의 `member`인지 여부를 알 수 있는 방법이 없기 때문입니다.

---

> 본 포스팅은 [`gRPC` 공식 문서](https://grpc.io/docs/what-is-grpc/introduction/)를 참고하여 작성하였습니다.