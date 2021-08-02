[comment]: <> (![]&#40;https://img.shields.io/badge/spring--boot-2.5.2-red&#41; ![]&#40;https://img.shields.io/badge/gradle-7.1.1-brightgreen&#41; ![]&#40;https://img.shields.io/badge/java-11-blue&#41;)

[comment]: <> (> 소스 코드는 [여기]&#40;https://github.com/lcalmsky/grpc&#41; 있습니다.)

`gRPC`(google Repote Procedure Call)란 구글에서 만든 원격 프로시저 호출 프레임워크 입니다.

꾸준히 업데이트 되고있는 오픈소스로 성능이 매우 좋고 어떠한 환경에서도 실행시킬 수 있다는 장점을 가지고 있어 이미 몇 년 전부터 각광받는 기술입니다.

로드 밸런싱, 추적(trace), 상태 확인 및 인증을 위한 플러그형 지원을 통해 서비스를 효율적으로 연결할 수 있습니다. 또한 디바이스, 모바일 애플리케이션 및 브라우저를 백엔드 서비스에 연결하기 위한 분산 컴퓨팅의 마지막 단계에도 적용할 수 있습니다.

강력한 바이너리 직렬화 도구이자 언어인 프로토콜 버퍼(Protocol Buffer)를 이용하여 서비스를 정의하고, 간결하게 런타임 및 개발 환경을 설치하고 프레임워크를 사용하여 초당 수백만 개의 RPC로 확장할 수 있는 스케일링 기능을 제공합니다. 그리고 다양한 언어 및 플랫폼에 대해 클라이언트 및 서버 스텁을 자동 생성해줄 뿐만 아니라 양방향 스트리밍 및 완전한 통합 인증을 HTTP/2 기반 전송을 통해 제공합니다.

현재 넷플릭스 등 많은 개발 회사에서 사용중인 기술이므로 이번 기회에 확실하게 알아두려고 포스팅을 결심하게 되었습니다.

### 개요

`gRPC`에서 클라이언트 애플리케이션은 로컬에서 객체를 호출하듯이 서버 애플리케이션의 메서드를 직접 호출할 수 있어 분산 응용 프로그램 및 서비스를 더 쉽게 만들 수 있습니다.

다른 RPC 시스템 같이 `gRPC`는 파라미터와 리턴 타입을 사용하여 원격으로 호출할 수 있는 메서드를 지정하여 서비스를 정의한다는 아이디어를 기반으로 합니다. 

서버 측에서 서버는 이 인터페이스를 구현하고 `gRPC` 서버를 실행하여 클라이언트 호출을 처리합니다.

클라이언트에는 서버와 동일한 방법을 제공하는 `Stub`이 존재합니다.

![gRPC 공식 문서 참조](https://grpc.io/img/landing-2.svg)
> [gRPC 공식 문서 참조](https://grpc.io/docs/what-is-grpc/introduction/)

`gRPC` 클라이언트와 서버는 `Google` 내부의 서버에서 사용자의 데스크톱에 이르기까지 다양한 환경에서 실행되고 서로 통신할 수 있으며 `gRPC`에서 지원하는 모든 언어로 작성할 수 있습니다.

예를 들어 `Go`, `Python` 또는 `Ruby`의 클라이언트를 사용하여 `Java`로 `gRPC` 서버를 쉽게 만들 수 있습니다.

또한 최신 `Google API`는 다양한 `gRPC` 버전의 인터페이스를 제공하므로 애플리케이션에 `Google`에서 제공하는 기능을 쉽게 구축할 수 있습니다.

### 프로토콜 버퍼(Protocol Buffers)

기본적으로 `gRPC`는 구조화된 데이터를 직렬화하기 위한 `Google`의 오픈 소스 메커니즘인 [프로토콜 버퍼](https://developers.google.com/protocol-buffers/docs/overview)를 사용합니다. (JSON과 같은 다른 데이터 포맷과도 함께 사용할 수 있습니다.)

작동 방식에 대해 간단히 소개합니다.

첫 번 째 단계로 `proto` 파일에서 직렬화하려는 데이터 구조를 정의합니다.

`.proto` 확장자를 가진 일반 텍스트 파일로, 프로토콜 버퍼 데이터는 메시지로 구성되며, 여기서 각 메시지는 필드라고 하는 일련의 이름-값 쌍을 포함하는 정보의 작은 논리적 레코드입니다.

예시를 살펴볼까요?

```
message Person {
  string name = 1;
  int32 id = 2;
  bool has_ponycopter = 3;
}
```

이런식으로 메시지(데이터 구조)를 정의하고 프로토콜 버퍼 컴파일러 `protoc`를 사용하여 원하는 언어로 데이터 클래스를 생성합니다.

자바 기준으로 `getName()`, `setName()`과 같은 각 필드에 대한 `getter/setter`와 원시 바이트에서 전체 구조를 직렬화하고 분석하는 메서드를 제공합니다.

위의 예제를 가지고 `protoc`를 실행하면 `Person`이라는 클래스가 생성되고 애플리케이션에서 이 클래스를 사용하여 프로토콜 버퍼 메시지를 채우고 직렬화하고 검색할 수 있습니다.

프로토콜 버퍼 메시지로 지정된 메서드 파라미터 및 리턴 타입을 사용하여 일반 `proto` 파일에서 `gRPC` 서비스를 정의합니다.

```
// The greeter service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

`gRPC`는 플러그인과 함께 `protoc`를 사용하여 `proto` 파일에서 코드를 생성합니다.

생성된 `gRPC` 클라이언트 및 서버 코드는 물론이고 메시지 타입을 채우고 직렬화하고 검색하기 위한 일반 프로토콜 버퍼 코드를 받습니다.

선택한 언어로 `gRPC` 플러그인을 사용하여 `protoc`를 설치하는 방법을 포함하여 프로토콜 버퍼에 대해서는 다음 포스팅에서 자세히 다루도록 하겠습니다.

### 프로토콜 버퍼 버전

`Proto3`은 현재 `Java`, `C++`, `Dart`, `Python`, `Objective-C`, `C#`, `Android Java`, `Ruby`, `JavaScript`, `golang` 언어에 대해 소스 코드 생성을 지원하고 다른 언어에 대해서도 개발중에 있습니다.

해당 `repository`는 [여기](https://github.com/protocolbuffers/protobuf/releases)를 참조하세요.

`proto3` 언어 가이드 및 각 언어에 대해 제공되는 [참조 문서](https://developers.google.com/protocol-buffers/docs/proto3)에서 자세한 내용을 확인할 수 있습니다. 참조 문서에는 `.proto` 파일 형식에 대한 공식 사양도 포함되어 있습니다.

일반적으로 `proto2`(현재 기본 프로토콜 버퍼 버전)를 사용할 수 있지만 `proto3`를 `gRPC`와 함께 사용하면 `gRPC` 지원 언어의 전체 범위를 사용할 수 있을 뿐만 아니라 `proto2` 클라이언트와 통신하는 호환성 문제를 피할 수 있으므로 `proto3` 버전을 사용하는 것을 권장합니다.

> 본 포스팅은 [`gRPC` 공식 문서](https://grpc.io/docs/what-is-grpc/introduction/)를 참고하여 작성하였습니다.

---

다음 포스팅에선 프로토콜 버퍼(`proto3`)에 대해 다루겠습니다.