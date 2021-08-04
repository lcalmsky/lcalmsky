## Enumerations

메시지 타입을 정의할 때 해당 필드 중 하나에 미리 정의된 값들의 리스트 중 하나만 포함되도록 할 수 있습니다.

예를 들어, corpus가 UNIVERSAL, WEB, IMAGES, LOCAL, NEWS, PRODUCTS 또는 VIDEO 중 하나일 때 SearchRequest에 대해 corpus 필드를 추가하려고 한다고 가정해 보겠습니다.

메시지를 정의할 때 열거형(enum)을 사용하면 매우 간단히 해결할 수 있습니다.

열거형 타입으로 지정된 필드는 해당 enum 중 하나로만 지정할 수 있습니다.

만약 다른 값으로 지정했을 경우 알 수 없는 필드로 취급하게 됩니다.

```
message SearchRequest {
  required string query = 1;
  optional int32 page_number = 2;
  optional int32 result_per_page = 3 [default = 10];
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  optional Corpus corpus = 4 [default = UNIVERSAL];
}
```

위 예시에서는 Corpus라는 enum을 정의한 뒤 해당 필드를 추가하였고 기본 값을 UNIVERSAL로 지정한 것을 알 수 있습니다.

다른 열거형 상수에 동일한 값을 할당하여 별칭(alias)을 정의할 수 있습니다.

이렇게 하려면 allow_alias 옵션을 true로 설정해야 하는데 만약 설정하지 않고 별칭을 사용할 경우 프로토콜 컴파일러가 오류 메시지를 생성합니다.

```
enum EnumAllowingAlias {
  option allow_alias = true;
  UNKNOWN = 0;
  STARTED = 1;
  RUNNING = 1;
}
enum EnumNotAllowingAlias {
  UNKNOWN = 0;
  STARTED = 1;
  // RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
}
```

Enumerator 상수는 32 비트 정수 범위에 포함되어야 합니다.

enum 값은 [varint encoding](https://developers.google.com/protocol-buffers/docs/encoding)을 사용하므로 음수 값은 비효율적이라 권장하지 않습니다.

> Varint는 하나 이상의 바이트를 사용하여 정수를 직렬화하는 방법입니다. 숫자가 작을수록 더 적은 수의 바이트를 사용합니다.

열거형은 메시지 정의 내에서 또는 외부에서 모두 사용할 수 있고 .proto 파일의 모든 메시지 정의에서 사용될 수 있습니다.

메시지 타입 내에 정의된 열거형 타입 접근을 위해선 '.'을 사용합니다.

```
MessageType.EnumType
```

.proto 파일에 대해 프로토콜 버퍼 컴파일러를 실행하면 Java, C++, Python 언어의 Enum 타입에 해당하는 코드를 생성해줍니다.

## 다른 메시지 타입 사용하기

다른 메시지 타입을 필드 타입으로 정의 할 수 있습니다.

예를 들어 SearchResponse라는 메시지에 Result라는 메시지 타입을 포함하고 싶다고 가정하면, .proto에 Result를 정의한 뒤 SearchResponse의 필드로 지정할 수 있습니다.

```
message SearchResponse {
  repeated Result result = 1;
}

message Result {
  required string url = 1;
  optional string title = 2;
  repeated string snippets = 3;
}
```

### 다른 메시지 정의 import 하기

위의 예시에서는 SearchResponse와 Result가 동일한 파일에 작성되어있습니다.

다른 .proto 파일에 정의되어 있다면 어떻게 해야 할까요?

보통 다른 언어들과 마찬가지로 다른 파일의 내용을 가져올 수 있는데 그 때 import를 사용합니다.

```
import "myproject/other_protos.proto";
```

.proto 파일의 최상단에 import 문을 작성해야 합니다.

기본적으로는 직접 import 한 파일의 정의만 사용할 수 있습니다.

이 경우 import 할 파일이 다른 위치로 이동하게 되면 파일을 직접 바꿔줘야하는 불편함이 있겠죠?

이 한 번의 변화에 대해 모든 호출하는 파일을 수정하는 하는 대신 더미 .proto 파일을 이전 경로에 위치시켜 새로운 위치를 가리킬 수 있게 하는데 이 때 import public을 사용합니다.

import public는 import public 문을 포함하는 proto를 import하는 모든 proto에 전이적으로 의존합니다.

영어를 번역하니 말이 너무 어려워졌는데 아래 예시를 보시면 이해하기 쉬울 겁니다.

*new.proto*
```
// All definitions are moved here
```

*old.proto*
```
// This is the proto that all clients are importing.
import public "new.proto";
import "other.proto";
```

*client.proto*
```
import "old.proto";
// You use definitions from old.proto and new.proto, but not other.proto
```

이전 proto 파일이 old.proto, 새로운 위치로 이동한 proto 파일이 new.proto이고, new.proto로 기존에 old.proto에 정의한 내용이 모두 이동했다고 가정해봅시다.

client.proto에서 old.proto를 import하고 있습니다.

old.proto에서는 import public을 이용해 new.proto를 불러오고 그냥 import를 사용해 다른 proto를 불러오고 있습니다.

이 경우 client.proto에서는 public으로 import한 new.proto만 import하여 사용할 수 있습니다.

원래 import 하던 것을 변경하는 대신 새로운 위치로 이동한 new.proto를 public으로 import하면 기존 정의들을 모두 변경할 필요가 없습니다.

프로토콜 컴파일러는 -I/ --proto_path 플래그를 사용하여 프로토콜 컴파일러 명령줄에 지정된 디렉터리 집합에서 가져온 파일을 검색합니다.

플래그가 지정되지 않은 경우 컴파일러가 호출된 디렉토리를 찾습니다.

일반적으로 --proto_path 플래그를 프로젝트의 루트로 설정하고 모든 import에 대해 정규화된 이름을 사용해야 합니다.

### proto2 - proto3 메시지 타입 사용하기

두 버전의 메시지는 서로 호환되기 때문에 import 해서 사용할 수 있습니다.

하지만 proto2의 열거형은 proto3에서 직접 사용할 수 없고 import 한 proto2 내에서만 사용 가능합니다.

---

> 본 포스팅은 [`gRPC` 공식 문서](https://grpc.io/docs/what-is-grpc/introduction/)를 참고하여 작성하였습니다.

