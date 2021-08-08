## Service 정의하기

RPC(Remote Procedure Call, 원격 프로시저 호출) 시스템에서 메시지 타입을 사용하려는 경우 `.proto` 파일에 `RPC` 서비스 인터페이스를 정의하면 프로토콜 버퍼 컴파일러가 선택한 언어로 서비스 인터페이스 코드와 스텁을 생성합니다.

예를 들어, `SearchRequest`를 받아 `SearchResponse`를 반환하는 메서드로 `RPC` 서비스를 정의하려는 경우 다음과 같이 `.proto` 파일에 정의할 수 있습니다.

```protobuf
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```

프로토콜 버퍼와 함께 사용할 수 있는 가장 간단한 `RPC` 시스템이 바로 `Google`에서 개발한 언어 및 플랫폼에 대해 중립적인 오픈 소스 `RPC` 시스템인 `gRPC`입니다.

`gRPC`는 특히 프로토콜 버퍼와 잘 작동하며 특별한 프로토콜 버퍼 컴파일러 플러그인을 사용하여 `.proto` 파일에서 직접 관련 `RPC` 코드를 생성할 수 있습니다.

`gRPC`를 사용하지 않으려면 자체 `RPC` 구현과 함께 프로토콜 버퍼를 사용할 수도 있습니다.

이에 대한 자세한 내용은 [`Proto2` 언어 가이드](https://developers.google.com/protocol-buffers/docs/proto#services)에서 확인할 수 있습니다.

또한 프로토콜 버퍼용 `RPC` 구현을 개발하기 위해 진행 중인 여러 3rd-party 프로젝트가 있습니다.

프로젝트에 대한 링크 목록은 [3rd-party 위키 페이지](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md)를 참조하세요.

## JSON 매핑하기

`Proto3`는 `JSON`의 표준 인코딩을 지원하므로 시스템 간에 데이터를 더 쉽게 공유할 수 있습니다.

인코딩은 아래 표에 타입 별로 설명되어 있습니다.

`JSON`으로 인코딩된 데이터에 값이 없거나 값이 `null`인 경우 프로토콜 버퍼로 구문 분석할 때 적절한 기본값으로 해석됩니다.

필드에 프로토콜 버퍼의 기본 값이 있는 경우 공간을 절약하기 위해 기본적으로 `JSON` 인코딩 데이터에서 필드가 생략됩니다.

구현은 `JSON` 인코딩 출력에서 기본값이 있는 필드를 내보내는 옵션을 제공할 수 있습니다.

|proto3|JSON|JSON example|Notes|
|---|---|---|---|
|message|object|{"fooBar": v, "g": null, …}|JSON 객체를 생성합니다.<br>메시지 필드 이름은 lowerCamelCase에 매핑되어 JSON 객체 키가 됩니다.<br>json_name 필드 옵션이 지정되면 지정된 값이 대신 키로 사용됩니다.<br>파서는 lowerCamelCase 이름(또는 json_name 옵션으로 지정된 이름)과 원래 proto 필드 이름을 모두 허용합니다.<br>null은 모든 필드 유형에 대해 허용되는 값이며 해당 필드 유형의 기본값으로 처리됩니다.|
|enum|string|"FOO_BAR"|proto에 지정된 열거형 값의 이름이 사용됩니다.<br>파서는 열거형 이름과 정수 값을 모두 허용합니다.|
|map<K,V>|object|{"k": v, …}|모든 키는 문자열로 변환됩니다.|
|repeated V|array|[v, …]|null은 빈 리스트로 ([]) 허용됩니다.|
|bool|true, false|true, false|
|string|string|"Hello World!"|
|bytes|base64 string|"YWJjMTIzIT8kKiYoKSctPUB+"|JSON 값은 패딩이 있는 표준 base64 인코딩을 사용하여 문자열로 인코딩된 데이터입니다.<br>표준 또는 URL 안전 base64 인코딩(패딩 유무 포함)이 허용됩니다.|
|int32, fixed32, uint32|number|1, -10, 0|JSON 값은 십진수입니다. 숫자 또는 문자열이 허용됩니다.|
|int64, fixed64, uint64|string|"1", "-10"|JSON 값은 10진수 문자열입니다. 숫자 또는 문자열이 허용됩니다.|
|float, double|number|1.1, -10.0, 0, "NaN", "Infinity"|JSON 값은 숫자 또는 특수 문자열 값 "NaN", "Infinity" 및 "-Infinity" 중 하나입니다.<br>숫자 또는 문자열이 허용됩니다.<br>지수 표기법도 허용됩니다.<br>-0은 0과 동일한 것으로 간주됩니다.|
|Any|object|{"@type": "url", "f": v, … }|Any에 특별한 JSON 매핑이 있는 값이 포함되어 있으면 {"@type": xxx, "value": yyy}와 같이 변환됩니다.<br>그렇지 않으면 값이 JSON 객체로 변환되고 "@type" 필드가 삽입되어 실제 데이터 타입을 나타냅니다.|
|Timestamp|string|"1972-01-01T10:00:20.021Z"|RFC 3339(Z-정규화 적용 및 0, 3, 6, 9 개의 소수 자릿수 사용)를 사용합니다. "Z" 이외의 오프셋도 허용됩니다.|
|Duration|string|"1.000340012s", "1s"|생성된 출력에는 필요한 정밀도에 따라 항상 0, 3, 6 또는 9개의 소수 자릿수와 접미사 "s"가 포함됩니다.<br>나노 초(nano-seconds) 정밀도를 사용하는한 어떤 소숫점 자릿수도 허용됩니다.<br>"s" 접미사를 사용해야 합니다.|
|Struct|object|{ … }|모든 JSON 객체|
|Wrapper types|various types|2, "2", "foo", true, "true", null, 0, …|데이터 변환 및 전송 중에 null이 허용되고 보존된다는 점을 제외하고 JSON에서 래핑된 기본 유형과 동일한 표현을 사용합니다.|
|FieldMask|string|"f.fooBar,h"|See field_mask.proto.(이렇게만 적혀있고 링크도 없네요🥲)|
|ListValue|array|[foo, bar, …]|
|Value|value| |모든 JSON 값, 자세한 내용은 [여기](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#google.protobuf.Value)를 참조하세요.|
|NullValue|null||JSON null|
|Empty|object|{}|빈 JSON 객체|

### JSON 옵션

`proto3`에서 `JSON`을 구현할 때 다음과 같은 옵션을 제공합니다.

* 기본 값이 있는 필드 생략: 기본 값이 있는 필드는 `proto3` `JSON` 결과물에서 기본적으로 생략 옵션이 적용됩니다. 이 행위와 결과 필드를 기본 값으로 재정의하는 옵션도 제공합니다.
* 알 수 없는 필드 무시: `Proto3` `JSON` 파서는 기본적으로 알 수 없는 필드를 거부해야 하지만 구문 분석에서 알 수 없는 필드를 무시하는 옵션을 제공할 수 있습니다.
* `LowerCamelCase` 이름 대신 `proto` 필드 이름 사용: 기본적으로 `proto3` `JSON` 프린터는 필드 이름을 `lowerCamelCase`로 변환하고 이를 `JSON` 이름으로 사용해야 합니다. `proto` 필드 이름을 `JSON` 이름으로 사용하는 옵션을 제공합니다. 변환된 `lowerCamelCase` 이름과 `proto` 필드 이름을 모두 허용하기 위해서는 `Proto3` `JSON` 파서가 필요합니다.
* 열거형 값을 문자열 대신 정수로 내보내기:  열거형 값의 이름은 기본적으로 JSON을 출력할 때 사용됩니다. 열거형 값의 숫자 값을 대신 사용하는 옵션을 제공합니다.

---

> 본 포스팅은 [`gRPC` 공식 문서](https://grpc.io/docs/what-is-grpc/introduction/)를 참고하여 작성하였습니다.