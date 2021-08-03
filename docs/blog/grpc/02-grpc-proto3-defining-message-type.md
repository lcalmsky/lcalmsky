먼저 간단한 예제를 살펴보겠습니다.

검색 요청에 쿼리 문자열, 결과 페이지 및 페이지 당 결과 수가 있는 검색 요청 메시지 형식을 정의하려고 한다고 가정해 보겠습니다.

다음은 메시지 유형을 정의하는 데 사용하는 `.proto` 파일입니다.

```
syntax = "proto3"; // (1)

message SearchRequest { // (2)
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```
> (1) 첫 번 째 줄은 `proto3` 문을 사용함을 의미합니다. 이렇게 명시하지 않으면 프로토콜 버퍼 컴파일러가 `proto2`를 기본으로 사용합니다. 반드시 첫 번 째 줄에 작성해야 합니다.      
> (2) `SearchRequest` 메시지는 세 개의 필드를 가지고 각 필드는 타입과 이름을 가집니다.

### 필드 타입 지정

위 예시에서는 모든 필드가 `scalar type`을 가집니다. `Java`로 따지면 `primitive` 타입이라고 생각하시면 됩니다. `Enumeration`이라든지 다른 메시지 타입을 가질 수 있습니다.

### 필드 번호 할당

메시지를 정의할 때 각 필드에는 고유의 번호를 할당할 수 있습니다.

이 번호는 메시지 바이너리 포맷에서 각각의 필드를 식별하는데 사용되기 때문에 메시지를 사용중인 경우 변경하면 안 됩니다.

1~15 사이의 번호를 식별하는데는 1 byte가 필요하고 16에서 2047 사이의 번호를 식별하는데는 2 bytes가 필요합니다.

따라서 메시지 내에 자주 사용되는 필드를 1~15번 내로 위치시키는 것이 좋고, 향후 자주 사용될 필드가 추가될 가능성이 있을 경우 1~15번 사이에 공간을 남겨둔 채로 이후 필드를 작성하는 것이 좋습니다.

지정할 수 있는 번호 중 가장 작은 번호는 1이고 가장 큰 번호는 2^29 - 1(536,870,911) 입니다.

19000번에서 19999번은 프로토콜 버퍼 구현을 위해 예약되어 있으므로 사용할 수 없습니다.

이 외에도 [예약된 필드 번호](https://developers.google.com/protocol-buffers/docs/proto3#reserved)는 사용할 수 없습니다.

### 필드 룰 지정

메시지 필드는 `singular` 또는 `repeated` 일 수 있습니다.

`singular`는 메시지가 0 또는 한 개의 필드를 가지는 것으로 `proto3` 문법의 기본 값 입니다. `repeated`는 0~N 개의 필드를 가질 수 있고 순서를 유지합니다.

`proto3`에서 스칼라 숫자 타입의 `repeated` 필드는 기본적으로 [압축 인코딩](https://developers.google.com/protocol-buffers/docs/encoding#packed)을 사용합니다.

### 메시지 타입 추가

`.proto` 파일에는 여러 메시지 타입을 정의할 수 있습니다.

관련 메시지 타입을 정의하는 경우에 유용합니다.

```
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

message SearchResponse {
 ...
}
```

이런식으로 관련있는 요청/응답 메시지를 하나의 파일에 정의할 수 있습니다.

### 주석 추가

```
/* SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response. */

message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // Which page number do we want?
  int32 result_per_page = 3;  // Number of results to return per page.
}
```

`/* */` 또는 `//` 를 사용해 주석을 추가할 수 있습니다.

### 예약 필드

필드를 완전히 제거하거나 주석 처리하여 메시지 타입을 업데이트하면 이후 사용자가 해당 필드 번호를 재사용할 수 있는데 이런 경우 나중에 이전 버전을 사용하다가 데이터가 오염되거나 버그가 발생할 수 있습니다.

이러한 일이 발생하지 않도록 하기 위해서는 삭제된 필드의 번호(및/또는 JSON 직렬화 문제를 일으킬 수도 있는 이름)를 예약하도록 지정하는 것입니다.

`reserved` 키워드를 사용하고 사용 예시는 아래와 같습니다.

```
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

이렇게 지정할 경우 이후 2, 15, 9, 10, 11 번의 필드 번호아 foo, bar라는 필드 이름은 사용할 수 없습니다.

### .proto 파일을 이용해 생성되는 파일

`.proto`에서 프로토콜 버퍼 컴파일러를 실행할 때 컴파일러는 필드 값 가져 오기 및 설정, 메시지 직렬화를 포함하여 파일에서 설명한 메시지 타입으로 작업하는 데 필요한 코드를 선택한 언어로 생성합니다.

Input/Output Stream에서 메시지를 분석합니다.

* C++: `.h`, `.cc` 파일 생성
* Java: `.java` 파일 생성
* Kotlin: `.kt` 파일 생성
* Python: 모듈 생성 후 런타임에 필요한 데이터 액세스 클래스 생성
* Go: `.pb.go` 파일 생성
* Ruby: `.rb` 파일 생성
* Objective-C: `pbobjc.h`, `pbobjc.m` 파일 생성
* C#: `.cs` 파일 생성
* Dart: `.pb.dart` 파일 생성

언어에 따른 튜토리얼과 [API 문서](https://developers.google.com/protocol-buffers/docs/reference/overview)를 제공합니다.

---

다음 포스팅에서는 `Scalar Value Type`과 기본 값에 대해 알아보겠습니다.

For C++, the compiler generates a .h and .cc file from each .proto, with a class for each message type described in your file.
For Java, the compiler generates a .java file with a class for each message type, as well as a special Builder classes for creating message class instances.
For Kotlin, in addition to the Java generated code, the compiler generates a .kt file for each message type, containing a DSL which can be used to simplify creating message instances.
Python is a little different – the Python compiler generates a module with a static descriptor of each message type in your .proto, which is then used with a metaclass to create the necessary Python data access class at runtime.
For Go, the compiler generates a .pb.go file with a type for each message type in your file.
For Ruby, the compiler generates a .rb file with a Ruby module containing your message types.
For Objective-C, the compiler generates a pbobjc.h and pbobjc.m file from each .proto, with a class for each message type described in your file.
For C#, the compiler generates a .cs file from each .proto, with a class for each message type described in your file.
For Dart, the compiler generates a .pb.dart file with a class for each message type in your file.