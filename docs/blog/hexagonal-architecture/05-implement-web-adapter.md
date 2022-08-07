> 이 포스팅은 [만들면서 배우는 클린 아키텍처](http://www.yes24.com/Product/Goods/105138479)를 읽고 작성하였습니다.  
> 소스 코드는 [여기](https://github.com/thombergs/buckpal.git) 있습니다.

## Overview

웹 인터페이스를 제공하는 어댑터의 구현 방법을 살펴봅니다.

## 의존성 역전

웹 어댑터는 주도하는(driving) 또는 인커밍 어댑터 입니다. 외부로부터 요청을 받아 애플리케이션 코어를 호출합니다. 이 때 제어 흐름은 웹 어댑터에 있는 컨트롤러에서 애플리케이션 계층에 있는 서비스로 흐릅니다.

애플리케이션 계층은 웹 어댑터가 통신할 수 있는 포트 인터페이스인 유스케이스를 제공하고 해당 유스케이스를 구현합니다.

여기 의존성 역전 원칙이 적용된 것을 확인할 수 있습니다.

![](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/lcalmsky/lcalmsky/master/docs/blog/hexagonal-architecture/05-01.puml)

어댑터와 유스케이스 사이에 인터페이스가 존재하는 이유는 애플리케이션 코어가 외부와 통신할 수 있는 규격이기 때문입니다. 이 포트를 적절하게 위치시키면 외부와 어떤 통신이 일어나는지 알 수 있고 레거시 코드를 다루는 경우 매우 소중한 정보가 됩니다.

우리는 익숙한 방식으로 인커밍 포트를 생략하고 컨트롤러에서 서비스를 바로 호출하는 지름길을 택하고 싶을 수 있습니다. 이 부분은 추후 다시 다루도록 하겠습니다.

상호작용이 많이 일어나는 애플리케이션의 경우 의문이 생길 수 있습니다. 웹소켓을 통해 실시간 데이터를 브라우저로 전송한다고 가정하면 애플리케이션 코어에서 어떻게 웹 어댑터로 전달할 수 있을까요?

이 시나리오에서는 반드시 포트가 필요합니다.

![](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/lcalmsky/lcalmsky/master/docs/blog/hexagonal-architecture/05-03.puml)

정확하게는 이 포트의 경우 아웃고잉 포트가 되기 때문에 웹 어댑터는 인커밍 어댑터인 동시에 아웃고잉 어댑터가 됩니다.

## 웹 어댑터의 책임

웹 어댑터의 책임은 다음과 같습니다.

1. HTTP 요청을 자바 객체로 매핑
2. 권한 검사
3. 입력 유효성 검증
4. 입력을 유스케이스 입력 모델로 매핑
5. 유스케이스 호출
6. 유세크이스의 출력을 HTTP로 매핑
7. HTTP 응답 반환

일반적으로 우리가 컨트롤러의 역할이라고 알고 있는 것과 크게 다른 점은 없습니다.

HTTP 요청을 수신한 뒤 객체로 역직렬화하고, 인증과 권한 부여를 수행하고 실패할 경우 에러를 반환합니다. 그 이후 요청에 대한 유효성을 검증한 뒤 유스케이스에서 사용할 모델로 매핑하여 전달합니다.

유스케이스에서 입력 모델을 검증했던 것과 조금 다른 성격인데, 유스케이스의 입력 모델로 변환할 수 있는지 여부를 검증한다고 표현하는 것이 정확합니다.

자연스럽게 다음 순서인 유스케이스 호출로 이어지고, 유스케이스의 출력을 전달받아 응답을 매핑하여 반환합니다.

이 과정에서 하나라도 문제가 생기면 예외를 던지고 호출한 쪽에 전달할 수 있는 에러 메시지로 변환하여 전달해야 합니다.

책임이 다소 많아보이긴 하지만 애플리케이션 계층에서 신경쓰면 안 되는 것들이기도 합니다.

웹 어댑터와 애플리케이션 계층간의 경계는 도메인과 애플리케이션 계층부터 개발하기 시작하면 자연스럽게 생성됩니다. 특정 인커밍 어댑터를 대상으로 구현하는 것이 아닌 오로지 유스케이스만 염두해두고 구현해야하기 때문입니다.

## 컨트롤러 나누기

스프링 MVC 같은 대부분의 웹 프레임워크에서는 앞서 논의한 책임들을 수행할 컨트롤러 클래스를 생성할 수 있습니다. 그리고 여태까지 해왔던 것처럼 모든 요청을 다룰 수 있는 하나의 컨트롤러를 생성할 수 있습니다. 하지만 웹 어댑터는 꼭 하나의 클래스로 구성할 필요가 없습니다.

클래스들은 같은 소속이라는 것을 표현하기 위해 패키지를 사용합니다. 그렇다면 컨트롤러를 몇 개를 만들어야 적당할까요? 너무 적은 것보다는 너무 많은 게 낫습니다. 각 컨트롤러가 좁고 다른 컨트롤러와 공유하는 게 적은 웹 어댑터 조각을 구현해야 합니다.

책의 저자는 각 연산에 대해 가급적이면 별도의 패키지 안에 별도의 컨트롤러를 만드는 방식을 선호한다고 밝히고 있습니다. 그리고 가급적 메서드와 클래스명은 유스케이스를 최대한 반영해야 한다고 하고 있습니다.

```java
package buckpal.adapter.web;

@RestController
@RequiredArgsConstructor
class SendMoneyController {

  private final SendMoneyUseCase sendMoneyUseCase;

  @PostMapping(path = "/accounts/send/{sourceAccountId}/{targetAccountId}/{amount}")
  void sendMoney(
      @PathVariable("sourceAccountId") Long sourceAccountId,
      @PathVariable("targetAccountId") Long targetAccountId,
      @PathVariable("amount") Long amount) {

    SendMoneyCommand command = new SendMoneyCommand(
        new AccountId(sourceAccountId),
        new AccountId(targetAccountId),
        Money.of(amount));

    sendMoneyUseCase.sendMoney(command);
  }
}
```

나눴을 때의 장점은 이미 유스케이스 때 이미 다뤘으므로 생략하고 넘어가도록 하겠습니다.

## 유지보수에 어떻게 도움이 될까?

웹 어댑터를 구현할 때 어떤 도메인 로직도 수행하지 않고 있습니다. 오직 웹 어댑터만의 책임을 다루므로 유스케이스에 맞게 어댑터를 교체하기 쉬워집니다.

웹 컨트롤러를 나눌 때 모델을 공유하지 않는 여러 작은 클래스를 만드는 것이 처음엔 많이 어색하겠지만, 작은 클래스로 구성하게 되면 더 파악하기 쉽고 테스트하기 쉬우며 동시에 작업하더라도 서로 충돌할 염려가 줄어듭니다.

처음에 이렇게 세분화시키는데는 공수가 더 들어갈 수 있지만 유지보수 단계에서는 빛을 발할 수 있습니다.