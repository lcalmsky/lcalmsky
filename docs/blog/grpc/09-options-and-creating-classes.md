## Option

`.proto` 파일 내에서 여러 가지 옵션을 사용할 수 있습니다.

옵션은 선언의 전체 의미를 변경하지 않지만 특정 컨텍스트에서 처리되는 방식에 영향을 줄 수 있습니다.

사용 가능한 옵션의 전체 목록은 `google/protobuf/descriptor.proto`에 정의되어 있습니다.

파일 레벨 옵션은 메시지나 열거형, 서비스 정의 안 쪽에 작성해서는 안 되고 반드시 최 상단에 작성해야 합니다.

메시지 레벨 옵션은 메시지 정의 안 쪽에 작성해야 합니다.

필드 레벨 옵션은 필드 정의 안 쪽에 작성해야 합니다.

옵션은 열거형 타입이나 값, oneof 필드, service 타입, service 메서드에 사용할 수 있습니다만 현재는 해당 레벨에 사용했을 때 유용한 옵션은 존재하지 않습니다.

가장 많이 사용되는 몇 가지 옵션을 살펴보겠습니다.

* java_package (file option)
  * 생성된 Java나 Kotlin 클래스에서 사용하기 위해 사용
  * .proto 파일에 java_package 옵션을 명시하지 않으면 기본 proto 패키지가 사용
  * proto 패키지는 도메인 역순 이름으로 생성되지 않기 때문에 일반적인 Java package를 생성하지 않음
  * Java나 Kotlin 코드를 생성하지 않을 경우 옵션은 무시됨
  ```protobuf
  option java_package = "com.example.foo";
  ```
* java_outer_classname (file option)
  * 생성하기 원하는 자바 클래스 래퍼에 대한 클래스 이름을 지정
  * .proto 파일에 java_outer_classname을 명시하지 않으면 클래스 이름은 .proto 파일에 명시된 이름을 camel-case 형태로 변환 (foo_bar.proto -> FooBar.java) 
  * java_multiple_files 옵션이 비활성화되면 .proto 파일에 대해 생성된 다른 모든 클래스(enum 등)가 외부 래퍼 Java 클래스 내에서 중첩된 클래스(enum 등)로 생성
  * 자바 코드를 생성하지 않을 경우 옵션은 무시됨
  ```protobuf
  option java_outer_classname = "Ponycopter";
  ```
* java_multiple_files (file option)
  * false로 지정할 경우 오직 하나의 .java 파일이 생성되고, top-level 메시지, 서비스, enum에 대해 생성된 모든 자바 클래스(enum 등)는 outer 클래스 내에 중첩됨
  * true로 지정할 경우 위와 같은 상황에 대해 각각의 .java 파일이 생성됨
  * 기본 옵션 값은 false
  * 자바 코드를 생성하지 않을 경우 옵션은 무시됨
  ```protobuf
  option java_multiple_files = true;
  ```
* optimize_for (file option)
  * SPEED, CODE_SIZE, LITE_RUNTIME 설정 가능하고 C++과 Java 코드 제너레이터에 적용 가능
  * SPEED(기본 값)
    * 프로토콜 버퍼 컴파일러가 메시지 타입에 대해 직렬화, 파싱, 기타 공통 옵션을 수행하며 코드를 고도로 최적화하여 생성
  * CODE_SIZE
    * 프로토콜 버퍼 컴파일러는 최소로 클래스를 생성하고 공유와 reflection 기반 코드에 의존하여 직렬화, 파싱 및 기타 다양한 작업을 구현
    * 생성된 코드는 SPEED에 비해 매우 크기가 작으나 수행시간이 느림
    * 생성된 클래스는 SPEED와 동일한 API 구현, 많은 수의 .proto 파일이 포함되어있고 빠르게 수행해야 할 필요가 없을 때 유용
  * LITE_RUNTIME
    * 프로토콜 버퍼 컴파일러는 lite runtime library를 이용해 클래스를 생성
    * lite runtime library는 full library에 비해 매우 작으나 descriptor나 reflection 같은 기능은 지원하지 않음
    * 모바일 환경에서 동작하는 앱에 유용
    * SPEED 모드에서와 같이 빠르게 구현
    * 생성된 클래스는 각 언어의 MessageLite 인터페이스만 구현하고 전체 Message 인터페이스의 메서드 하위 집합만 제공
  ```protobuf
  option optimize_for = CODE_SIZE;
  ```
* cc_enable_arenas (file option)
  * C++ 생성 코드에 대한 arena allocation 활성화
* objc_class_prefix (file option)
  * .proto에서 생성된 모든 Objective-C 클래스 및 열거형 앞에 Objective-C 클래스 접두사를 설정
  * 기본 값은 없음
  * Apple에서 권장하는 대로 3-5개의 대문자로 된 접두사를 사용(두 글자 접두사는 Apple에서 이미 예약하여 사용중)
* deprecated (field option)
  * true로 설정하면 필드가 더 이상 사용되지 않음을 나타냄(새 코드에서 사용하면 안 됨)
  * 대부분의 언어에서 실제로 효과가 없고 IDE에서 컴파일러가 warning으로 알려줌
  * 필드가 다른 사람에 의해 사용되지 않고, 새 사용자가 사용하는 것을 방지하기 위해서는 필드 선언을 reserved로 바꾸는 것을 권장
  ```protobuf
  int32 old_field = 6 [deprecated = true];
  ```

### 사용자 옵션

프로토콜 버퍼를 사용하면 고유한 옵션을 정의하고 사용할 수도 있습니다.

이것은 대부분의 사람들에게는 필요하지 않은 고급 기능입니다.

자신만의 옵션을 만들어야 한다고 생각한다면 자세한 내용은 [Proto2 언어 가이드](https://developers.google.com/protocol-buffers/docs/proto#customoptions)를 참조하세요.

사용자 지정 옵션을 만들 때는 proto3의 사용자 지정 옵션에만 허용되는 [Extensions](https://developers.google.com/protocol-buffers/docs/proto#extensions)을 사용합니다.

## 클래스 생성

.proto 파일에 정의된 메시지 타입으로 작업해야 하는 Java, Kotlin, Python, C++, Go, Ruby, Objective-C, C 코드를 생성하려면 .proto에서 프로토콜 버퍼 컴파일러 protoc를 실행해야 합니다.

만약에 컴파일러를 설치하지 않았다면 [패키지를 다운로드](https://developers.google.com/protocol-buffers/docs/downloads)하고 README의 지침을 따라 설치하시면 됩니다.

Go의 경우 컴파일러용 특수 코드 생성기 플러그인도 설치해야 합니다. GitHub의 [golang/protobuf 저장소](https://github.com/golang/protobuf/)에서 설치 방법을 확인할 수 있습니다.

프로토콜 컴파일러는 다음과 같이 호출됩니다.

```protobuf
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

* IMPORT_PATH
  * .proto 파일을 찾을 디렉토리를 지정
  * 생략할 경우 현재 디렉토리 사용
  * --proto_path 옵션을 여러 번 사용하여 여러 디렉토리 지정 가능
  * 순서대로 검색을 수행
  * -I로 사용 가능
* output에 대해 하나 이상의 옵션을 적용할 수 있음
  * --cpp_out: C++ 코드를 DST_DIR 안에 생성([참조](https://developers.google.com/protocol-buffers/docs/reference/cpp-generated))
  * --java_out: Java 코드를 DST_DIR 안에 생성([참조](https://developers.google.com/protocol-buffers/docs/reference/java-generated))
  * --kotlin_out: Kotlin 코드를 DST_DIR 안에 생성([참조](https://developers.google.com/protocol-buffers/docs/reference/kotlin-generated))
  * --python_out: Python 코드를 DST_DIR 안에 생성([참조](https://developers.google.com/protocol-buffers/docs/reference/python-generated))
  * --go_out: Go 코드를 DST_DIR 안에 생성([참조](https://developers.google.com/protocol-buffers/docs/reference/go-generated))
  * --ruby_out: Ruby 코드를 DST_DIR 안에 생성
  * --objc_out: Objective-C 코드를 DST_DIR 안에 생성([참조](https://developers.google.com/protocol-buffers/docs/reference/objective-c-generated))
  * --csharp_out: C# 코드를 DST_DIR 안에 생성([참조](https://developers.google.com/protocol-buffers/docs/reference/csharp-generated))
  * --php_out: PHP 코드를 DST_DIR 안에 생성([참조](https://developers.google.com/protocol-buffers/docs/reference/php-generated)), 추가 편의를 위해 DST_DIR이 .zip 또는 .jar로 끝나는 경우 컴파일러는 지정된 이름의 단일 zip 형식 아카이브 파일을 작성, .jar 출력에는 Java JAR 사양에서 요구하는 대로 매니페스트 파일 제공, 이미 있는 경우 덮어씀
* 하나 이상의 .proto 파일을 입력으로 제공해야 함
  * 여러 .proto 파일을 한 번에 지정할 수 있음
  * 파일의 이름은 현재 디렉터리를 기준으로 지정되지만 각 파일은 IMPORT_PATH 중 하나에 있어야 컴파일러가 정식 이름을 결정할 수 있음

---

> 본 포스팅은 [`gRPC` 공식 문서](https://grpc.io/docs/what-is-grpc/introduction/)를 참고하여 작성하였습니다.

다음 부터는 드디어 gRPC를 적용해서 개발을 해 볼 수 있겠네요.

일단 생각하고 있는 건 자바 서버-클라이언트 통신을 gRPC를 이용해 구현해보려고 합니다.
(제가 다른 언어는 아무 것도 몰라서😭)