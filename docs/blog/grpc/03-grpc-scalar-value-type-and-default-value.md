## 스칼라 데이터 타입(Scalar Value Type)

스칼라 메시지 필드는 아래 타입 중 하나를 가질 수 있습니다.

아래 테이블은 `.proto` 파일에 지정된 타입과 자동으로 생성된 클래스의 타입을 보여줍니다.

|.proty Type|Notes|C++ Type|Java/Kotlin Type|Python Type|Go Type|Ruby Type|C# Type|PHP Type|Dart Type|
|---|---|---|---|---|---|---|---|---|---|
|double||double|double|float|float64|Float|double|float|double|
|float||float|float|float|float32|Float|float|float|double|
|int32|가변 길이 인코딩을 사용, 음수 인코딩에 비효율적, 음수 가능성이 있는 경우 sint32 사용|int32|int|int|int32|Fixnum or Bignum (as required)|int|integer|int|
|int64|가변 길이 인코딩을 사용, 음수 인코딩에 비효율적, 음수 가능성이 있는 경우 sint64 사용|int64|long|int/long[4]|int64|Bignum|long|integer/string[6]|Int64|
|uint32|가변 길이 인코딩을|uint32|int[2]|int/long[4]|uint32|Fixnum or Bignum (as required)|uint|integer|int|
|uint64|가변 길이 인코딩을|uint64|long[2]|int/long[4]|uint64|Bignum|ulong|integer/string[6]|Int64|
|sint32|가변 길이 인코딩을, 부호 있는 정수 값, int32보다 음수를 더 효율적으로 인코딩|int32|int|int|int32|Fixnum or Bignum (as required)|int|integer|int|
|sint64|가변 길이 인코딩을, 부호 있는 정수 값, int64보다 음수를 더 효율적으로 인코딩|int64|long|int/long[4]|int64|Bignum|long|integer/string[6]|Int64|
|fixed32|항상 4 바이트, 값이 2^28보다 큰 경우 uint32보다 효율적으로 인코딩|uint32|int[2]|int/long[4]|uint32|Fixnum or Bignum (as required)|uint|integer|int|
|fixed64|항상 8 바이트, 값이 2^56보다 큰 경우 uint32보다 효율적으로 인코딩|uint64|long[2]|int/long[4]|uint64|Bignum|ulong|integer/string[6]|Int64|
|sfixed32|항상 4 바이트|int32|int|int|int32|Fixnum or Bignum (as required)|int|integer|int|
|sfixed64|항상 8 바이트|int64|long|int/long[4]|int64|Bignum|long|integer/string[6]|Int64|
|bool||bool|boolean|bool|bool|TrueClass/FalseClass|bool|boolean|bool|
|string|문자열은 항상 UTF-8 인코딩 이거나 7비트 ASCII 텍스트를 포함해야 함, 2^32보다 길 수 없음|string|String|str/unicode[5]|string|String (UTF-8)|string|string|String|
|bytes|2^32 이하의 임의의 바이트 시퀀스를 포함 가능|string|ByteString|str|[]byte|String (ASCII-8BIT)|ByteString|string|List|

프로토콜 버퍼 인코딩에서 메시지를 직렬화할 때 위에 명시된 타입들이 인코딩되는 방법에 대해 자세히 알아볼 수 있습니다.

[1] `Kotlin`은 `unsigned` 타입의 경우에도 `Java`에 해당하는 타입을 사용합니다. `Java/Kotlin`간 코드 호환성을 보장하기 위함입니다.  

[2] `Java`에서 부호 없는 32비트 및 64비트 정수는 부호 있는 정수를 사용하여 표현되며 최상위 비트는 단순히 부호 비트에 저장됩니다.

[3] 모든 경우에 필드에 값을 설정하면 타입 검사를 수행하여 유효한지 확인합니다.

[4] 64비트 또는 `unsigned` 32비트 정수는 디코딩 시 항상 `long`으로 표시되지만 필드를 설정할 때 int가 지정되면 int가 될 수 있습니다. 값은 모든 경우에 설정 시 지정한 타입에 맞아야 합니다. ([2] 참조)

[5] `Python` 문자열은 디코딩 시 유니코드로 표시되지만 `ASCII` 문자열이 제공되면 `str`이 될 수 있습니다(변경될 수 있음).

[6] 64비트 시스템에서는 정수가 사용되고 32비트 시스템에서는 문자열이 사용됩니다.

## 기본 값(Default Values)

메시지를 파싱할 때 인코딩된 메시지에 특정 단일 요소가 포함되어 있지 않으면, 파싱된 객체의 해당하는 필는 필드에 대한 기본 값이 설정됩니다.

기본 값은 타입 별로 다릅니다.

* 문자열: 빈 문자열
* byte: 빈 바이트
* bool: false
* numeric: 0
* enums: 첫 번 째 enum 값, ordinal 값은 0
* 메시지 필드: 설정되지 않음

`repeated` 필드의 경우 비어있습니다. 일반 적으로 `empty list`에 해당합니다.

각 언어 별로 빈 문자열이나 빈 바이트, 객체 등을 나타내는 문법이 다릅니다.

스칼라 메시지 필드의 경우 메시지가 파싱되면 필드가 명시적으로 기본값으로 설정되었는지 여부를 알 수 있는 방법이 없습니다.

따라서 메시지 타입을 정의할 때 각별히 주의해야 합니다.

또한 스칼라 메시지 필드가 기본 값으로 설정된 경우 값이 직렬화되지 않습니다.

생성된 코드에서 기본 값이 작동하는 방식에 대한 자세한 내용은 선택한 언어에 대해 [생성된 코드 가이드](https://developers.google.com/protocol-buffers/docs/reference/overview)를 참조하시면 됩니다.

---

다음 포스팅에서는 Enumeration과 메시지 타입 내에 다른 타입을 사용하는 방법에 대해 다뤄보겠습니다.

> 본 포스팅은 [`gRPC` 공식 문서](https://grpc.io/docs/what-is-grpc/introduction/)를 참고하여 작성하였습니다.


