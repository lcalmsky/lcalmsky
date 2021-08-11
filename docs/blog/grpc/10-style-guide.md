개발에 앞서 컨벤션충인 저는 `.proto` 파일 작성에 필요한 컨벤션을 가볍게 알아보겠습니다.

관련 정보는 공식 문서 내 [스타일 가이드](https://developers.google.com/protocol-buffers/docs/style)를 참고하였습니다.

---

이 문서는 `.proto` 파일에 대한 스타일 가이드를 제공합니다.

컨벤션을 따르면 프로토콜 버퍼 메시지 정의와 해당 클래스를 일관되고 읽기 쉽게 만들 수 있습니다.

> 프로토콜 버퍼 스타일은 시간이 지남에 따라 발전했기 때문에 다른 규칙이나 스타일로 작성된 .proto 파일을 볼 수 있습니다. 이러한 레거시 파일을 수정할 때는 기존 스타일을 준수하는 것이 좋습니다. 뭐든 일관성이 있는 게 중요하니까요. 일관성이 핵심입니다. 그러나 새 .proto 파일을 만들 때는 현재 최상의 스타일을 적용하는 것이 가장 좋습니다.

---

### 파일 포매팅

* 줄 길이를 80자로 유지하세요.
* 2칸 들여쓰기를 사용하세요.
* 문자열에는 큰따옴표를 사용하세요.

### 파일 구조

파일 이름은 `lower_snake_case`를 사용하고 확장자는 `.proto` 여야 합니다.

모든 파일은 다음과 같은 방식의 순서를 가져야 합니다.

1. 라이선스 헤더(해당되는 경우)
2. 파일 개요
3. 문법(어떤 버전을 사용할지) 
4. package
5. import (정렬 필요)
6. File options 
7. 그 밖에 필요한 것들

### 패키지

패키지 이름은 소문자여야 하며 디렉터리 계층 구조와 일치해야 합니다.

예를 들어 파일이 my/package에 있으면 패키지 이름은 my.package여야 합니다.

### 메시지와 필드 이름

메시지 이름에 CamelCase(초기 대문자 포함)를 사용합니다(예: SongServerRequest).

필드 이름(필드 및 확장 이름 중 하나 포함)은 underscore_separated_names를 사용합니다(예: song_name).

```protobuf
message SongServerRequest {
  optional string song_name = 1;
}
```

필드 이름에 이 명명 규칙을 사용하면 다음과 같은 접근자가 제공됩니다.

* C++
```c++
  const string& song_name() { ... }
  void set_song_name(const string& x) { ... }
```
* Java
```java
  public String getSongName() { ... }
  public Builder setSongName(String v) { ... }
```

필드 이름에 숫자가 포함된 경우 숫자는 밑줄 대신 문자 뒤에 나타나야 합니다(예: song_name_1 대신 song_name1 사용).

### repeated 필드

`repeated` 필드에는 복수형 이름을 사용합니다.

```protobuf
repeated string keys = 1;
repeated MyMessage accounts = 17;
```

### enum

열거형 이름에는 CamelCase(초기 대문자 포함)를 사용하고 값 이름에는 CAPITALS_WITH_UNDERSCORES를 사용합니다.

```protobuf
enum FooBar {
  FOO_BAR_UNSPECIFIED = 0;
  FOO_BAR_FIRST_VALUE = 1;
  FOO_BAR_SECOND_VALUE = 2;
}
```

각 열거형 값은 쉼표가 아닌 세미콜론으로 끝나야 합니다. enum 값을 둘러싸는 메시지로 묶는 대신 접두사를 사용하는 것을 선호합니다.

값이 0인 열거형에는 UNSPECIFIED 접미사가 있어야 합니다.

### service

서비스 이름과 RPC 메서드 이름 모두에 CamelCase(초기 대문자 포함)를 사용해야 합니다.

```protobuf
service FooService {
  rpc GetSomething(FooRequest) returns (FooResponse);
}
```