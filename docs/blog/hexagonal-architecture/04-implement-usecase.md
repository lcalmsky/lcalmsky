> 이 포스팅은 [만들면서 배우는 클린 아키텍처](http://www.yes24.com/Product/Goods/105138479)를 읽고 작성하였습니다.  
> 소스 코드는 [여기](https://github.com/thombergs/buckpal.git) 있습니다.

## Overview

앞서 살펴본 내용들을 어떻게 실제 코드로 구현할지 살펴봅니다.

## 도메인 모델 구현

한 계좌에서 다른 계좌로 송금하는 유스케이스를 구현합니다.

객체지향적으로 모델링하기 위해 입출금을 할 수 있는 Account 엔터티를 만들고, 출금 계좌에서 출금하여 입금 계좌로 입금하도록 하였습니다.

```java
package buckpal.domain;

@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class Account {

  @Getter
  private final AccountId id;
  @Getter
  private final Money baselineBalance;
  @Getter
  private final ActivityWindow activityWindow;
  
  // 생성자 생략
  
  public Money calculateBalance() {
    return Money.add(
        this.baselineBalance,
        this.activityWindow.calculateBalance(this.id));
  }

  public boolean withdraw(Money money, AccountId targetAccountId) {

    if (!mayWithdraw(money)) {
      return false;
    }

    Activity withdrawal = new Activity(
        this.id,
        this.id,
        targetAccountId,
        LocalDateTime.now(),
        money);
    this.activityWindow.addActivity(withdrawal);
    return true;
  }

  private boolean mayWithdraw(Money money) {
    return Money.add(
            this.calculateBalance(),
            money.negate())
        .isPositiveOrZero();
  }

  public boolean deposit(Money money, AccountId sourceAccountId) {
    Activity deposit = new Activity(
        this.id,
        sourceAccountId,
        this.id,
        LocalDateTime.now(),
        money);
    this.activityWindow.addActivity(deposit);
    return true;
  }
}
```

`Account` 엔터티는 실제 계좌의 현재 상태를 제공하고 계좌에 대한 모든 입출금 활동은 `Activity` 엔터티가 다룹니다. 한 계좌의 모든 내용을 다루는 것은 비효율적이기 때문에 특정 기간에 대한 활동을 `ActivityWindow`가 가지고 있습니다.

계좌의 잔고 계산을 위해 `Account`는 잔고(baselineBalance)를 속성으로 가지고 있고, 총 잔고는 잔고와 `ActivityWindow`의 모든 활동의 잔고를 합친 값이 됩니다.

입출금 기능을 가진 `Account` 엔터티가 있으므로 이를 중심으로 유스케이스를 구현할 수 있습니다.

## 유스케이스 둘러보기

유스케이스가 실제로 무슨 일을 하는지 살펴봅니다.

먼저 단계를 살펴보면,

1. 입력을 받는다.
2. 비즈니스 규칙을 검증한다.
3. 모델 상태를 변경한다.
4. 출력을 반환한다.

이런 순서가 됩니다.

인커밍 어댑터로부터 입력을 받는데 이 때 유효성 검증은 어댑터의 책임이므로 여기선 다루지 않습니다. 유스케이스는 오직 도메인 로직에만 신경쓸 수 있게 합니다.

유효성 검증은 하지 않지만 비즈니스 규칙에 해당하는 검증은 필요합니다.

비즈니스 규칙을 충족하는 경우 입력을 기반으로 모델의 상태를 변경합니다. 도메인 객체의 상태를 바꾸고 영속성 포트를 구현한 아웃고잉 어댑터를 이용해 상태를 업데이트 할 수 있게 합니다.

마지막으로 유스케이스를 호출한 어댑터 쪽으로 출력을 만들어 반환하면 됩니다.

여기서는 서비스가 비대해지는 것을 방지하기 위해 유스케이스별로 분리된 서비스를 구현합니다.

```java
package buckpal.application.service;

@RequiredArgsConstructor
@Transactional
public class SendMoneyService implements SendMoneyUseCase {

  private final LoadAccountPort loadAccountPort;
  private final AccountLock accountLock;
  private final UpdateAccountStatePort updateAccountStatePort;

  @Override
  public boolean sendMoney(SendMoneyCommand command) {
    // 비즈니스 규칙 검증
    // 모델 상태 변경
    // 출력 값 반환
  }
}
```

서비스는 인커밍 포트 인터페이스인 `SendMoneyUseCase`를 구현하고, 아웃커밍 포트 인터페이스인 `LoadAccountPort`를 호출합니다. 그리고 계좌 상태 업데이트를 위해 `UpdateAccountStatePort`를 호출합니다.

![](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/lcalmsky/lcalmsky/master/docs/blog/hexagonal-architecture/inport-service-outport.puml)

## 입력 유효성 검증

입력 유효성 검증 자체는 유스케이스의 책임이 아니긴 하지만, 여전히 애플리케이션 계층이 책임져야 하므로 이 챕터에서 다루고 넘어가려고 합니다.

호출하는 어댑터 쪽에서 유효성 검증을 하게 되면, 해당 유스케이스를 호출하는 모든 어댑터가 해당 검증을 구현해야하므로 그 과정에서 실수가 유발될 수 있고, 유효성 검증이 필요하다는 사실 자체를 까먹을 수 있기 때문입니다.

애플리케이션 계층을 호출할 당시에는 유효한 입력값이 전달되어 모델의 상태를 해치지 않게 해야합니다.

이 문제는 입력 모델이 다루도록 하면 됩니다.

```java
package buckpal.application.port.in;

@Getter
public class SendMoneyCommand extends SelfValidating<SendMoneyCommand> {

  private final AccountId sourceAccountId;
  private final AccountId targetAccountId;
  private final Money money;

  public SendMoneyCommand(
      AccountId sourceAccountId,
      AccountId targetAccountId,
      Money money) {
    this.sourceAccountId = sourceAccountId;
    this.targetAccountId = targetAccountId;
    this.money = money;
    requireNonNull(sourceAccountId);
    requireNonNull(targetAccountId);
    requireNonNull(money);
    requireGreaterThan(money, 0);
  }
}
```

패키지를 보시면 유스케이스 API의 일부이기 때문에 인커밍 포트 패키지(port.in)에 구현되어 있습니다. 애플리케이션 코어를 오염시키지 않고 검증할 수 있는 위치입니다. 

송금을 위해서는 출금, 입금 계좌의 ID, 송금할 금액이 필요하므로 모든 파라미터가 null이 아니어야하고 송금액은 0보다 커야합니다. 이 중 하나라도 위배될 때 생성자에서 예외를 던저 객체 생성을 막습니다.

모든 필드에 final을 붙여 객체 생성에 성공한 유효한 객체의 경우 값을 변경할 수 없다는 사실을 보장합니다.

이 코드는 validation 패키지를 활용하면 더 간단해집니다.

```java
package io.reflectoring.buckpal.account.application.port.in;

import io.reflectoring.buckpal.account.domain.Account.AccountId;
import io.reflectoring.buckpal.account.domain.Money;
import io.reflectoring.buckpal.common.SelfValidating;
import lombok.EqualsAndHashCode;
import lombok.Value;

import javax.validation.constraints.NotNull;

@Value
@EqualsAndHashCode(callSuper = false)
public class SendMoneyCommand extends SelfValidating<SendMoneyCommand> {

  @NotNull
  private final AccountId sourceAccountId;

  @NotNull
  private final AccountId targetAccountId;

  @NotNull
  private final Money money;

  public SendMoneyCommand(
      AccountId sourceAccountId,
      AccountId targetAccountId,
      Money money) {
    this.sourceAccountId = sourceAccountId;
    this.targetAccountId = targetAccountId;
    this.money = money;
    this.validateSelf();
  }
}
```

```java
package shared;

public abstract class SelfValidating<T> {

  private Validator validator;

  public SelfValidating() {
    ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
    validator = factory.getValidator();
  }

  protected void validateSelf() {
    Set<ConstraintViolation<T>> violations = validator.validate((T) this);
    if (!violations.isEmpty()) {
      throw new ConstraintViolationException(violations);
    }
  }
}
```

스프링을 사용할 때 컨트롤러에서 파라미터로 전달되는 요청에 @Valid를 붙이면 해당 요청과 매핑되는 클래스 내의 애너테이션이 동작하게 되는데, 같은 원리로 동작시킬 수 있도록 validator 클래스를 구현한 것입니다.

여기까지 입력 모델의 유효성 검증을 통해 유스케이스 주변에 오류를 방지할 수 있는 계층을 만들었습니다.

## 생성자의 힘

입력 모델인 `SendMoneyCommand`는 생성자가 많은 책임을 가지고 있습니다. final을 이용해 불변 클래스를 만들었고 생성자 내부에서 유효성 검증을 하기 때문에 유효하지 않은 상태의 객체를 만드는 것은 불가능해졌습니다.

현재는 필드가 세 개밖에 없기 때문에 큰 문제가 되지 않지만 파라미터가 20개인 경우엔 어떻게 해야할까요?

우리는 이미 @Builder 애너테이션을 활용하는 등 아주 쉽게 빌더 패턴을 이용하고 있습니다.

하지만 편리하게 사용할 수 있는 빌더 패턴은 우리가 열심히 작성한 생성자를 그냥 호출해주지 않기 때문에 유효하지 않은 객체를 생성할 가능성을 열어두게 됩니다.

책에서는 IDE를 믿고 생성자의 파라미터가 많더라도 그대로 사용할 것을 권장하고 있습니다.

## 유스케이스 별 입력 모델

다른 유스케이스에 동일한 입력 모델을 사용해도 될까요?

계좌 등록과 계좌 업데이트하기를 생각해보면, 두 입력 모델의 차이 점은 업데이트 할 때만 ID를 전달한다는 것입니다. 만약 같은 입력모델을 공유한다면 등록시에 ID 필드에 null을 전달해야 합니다.

불변 객체를 생성하면서 null로 초기화 한다는 것은 그 자체로 지독한 코드 냄새를 유발할 수 있고, 예외를 두기 시작한 유효성 검증 코드는 어딘가에서 또 검증해줘야하고 이는 관심사 오염으로 이어집니다.

따라서 유사한 유스케이스가 존재하더라도 각 유스케이스별로 입력 모델을 따로 정의하여 다른 유스케이스와의 결합도를 제거해야 합니다.

## 비즈니스 규칙 검증

앞서 다뤘던 입력 유효성 검증은 유스케이스의 책임이 아닌 반면에 비즈니스 규칙을 검증하는 것은 분명하게 유스케이스의 책임 중 일부라고 할 수 있습니다. 비즈니스 규칙은 애플리케이션의 핵심이기 때문입니다.

입력 유효성 검증과 비즈니스 규칙을 구분하는 방법은 도메인 모델의 현재 상태에 접근해야하는지로 나눌 수 있습니다. 입력 유효성 검증시에는 도메인 모델의 상태를 알 필요가 없고 접근할 필요가 없지만 비즈니스 규칙을 검증할 땐 반드시 상태를 알기위해 접근해야 하기 떄문입니다.

입력 유효성을 검증하는 것은 syntactical(구문상의, 문법의)한 검증이라고 할 수 있고, 비즈니스 규칙은 semantical(의미적인)한 검증이라고 할 수 있습니다.

"출금 계좌는 초과 출금되어서는 안 된다"라는 규칙은 출금 계좌의 잔고를 확인해야하기 때문에 비즈니스 규칙입니다. 반대로 "송금할 금액은 0보다 커야 한다"는 모델에 접근하지 않아도 검증할 수 있기 때문에 유효성 검증입니다.

비즈니스 검증의 위치는 바로 애플리케이션 레이어의 도메인 엔터티 안 쪽입니다.

```java
package buckpal.domain;

public class Account {
  // 생략
  public boolean withdraw(Money money, AccountId targetAccountId) {
    // 비즈니스 규칙 검증
    if (!mayWithdraw(money)) {
      return false;
    }
    // 생략
  }
}
```

비즈니스 로직에 바로 위치하고 있기 때문에 위치를 정하는 것도, 추론하는 것도 쉬워집니다.

도메인 엔티티에서 검증하는 것이 꺼려진다면 유스케이스 구현체 부분에서 비즈니스 로직 실행 전에 검사해도 됩니다.

```java
package buckpal.application.service;

public class SendMoneyService implements SendMoneyUseCase {
  // 생략
  @Override
  public boolean sendMoney(SendMoneyCommand command) {
    requireAccountExists(command.getSourceAccountId());
    requireAccountExists(command.getTargetAccountId());
    // 생략
  }
  // 생략
}
```

유효성 검증 코드를 먼저 호출하고 실패할 경우 예외를 던집니다. 인커밍 어댑터에서 이 예외를 캐치해서 사용자에게 보여줄 수 있는 에러로 변환하거나 다른 적절한 조치를 취하면 됩니다.

## 풍부한 도메인 모델 vs. 빈약한 도메인 모델

책에서 권장하는 아키텍처 스타일 중 도메인 모델을 구현하는 방법에 대해서는 열려있습니다. 어떻게 보면 상황에 맞게 해야하기 때문에 어쩔 수 없기도 하지만, 정확하게 제한이 되어있지 않으면 프레임워크로써 합격점을 줄 수 있을지는 의문이긴 합니다.

풍부한 도메인 모델인 DDD로 구현할 것인지, 빈약한 도메인 모델로 구현할 것인지에 대해 어느 하나를 선호하진 않지만 각각 어떤 상황에서 어떤 방식이 더 잘 어울리는지 소개합니다.

풍부한 도메인 모델에서는 애플리케이션 코어 내 엔터티에 가능한 많은 도메인 로직이 구현됩니다. 엔터티 상태를 변경한다든지, 비즈니스 규칙에 맞는 유효한 변경을 허용합니다. 이 경우 유스케이스는 도메인 모델의 진입점으로 동작하게 됩니다. 유스케이스는 사용자의 의도만을 표현하고 실제 동작은 도메인 엔터티를 호출하므로 비즈니스 규칙이 도메인 엔터티에 위치하게 됩니다.

빈약한 도메인 모델에서는 엔티티 자체가 도메인 로직을 가지지 않고 상태를 표현하는 필드와 값을 바꾸기 위한 메서드만 제공합니다. 이 경우 유스케이스에 도메인 로직이 구현됩니다. 비즈니스 규칙을 검증하고, 엔터티 상태를 바꾸고, 저장하는 일 모드 유스케이스가 주도하게 됩니다.

이 두 방법에 대해 정확한 정답은 없으니 개인이 더 선호하는 쪽 또는 팀에서 정한 방향으로 개발하시면 됩니다.

## 유스케이스 별 출력 모델

유스케이스가 할 일을 다 하면 호출한 인커밍 어댑터로 무엇을 반환해야 할까요?

입력 모델과 마찬가지로 출력 모델도 유스케이스별로 구체적인 게 좋습니다. 꼭 필요한 데이터만 구현할 수록 코드에서 나쁜 냄새가 나거나 유해한 부분이 전염되지 않습니다.

"송금하기" 유스케이스에서는 boolean 값 하나를 반환합니다. 보통 클라이언트에서 요청할 때 이미 입/출금 계좌 ID를 알고있고, 금액을 알고있음에도 결과 응답으로 이런 부가적인 것들을 추가해주는 경우가 있는데, 이 값이 호출한 쪽에 정말 필요한지는 생각해봐야 합니다. 필요 없는데 추가해놓은 경우, 이후 다른 곳에서 다른 의도로 호출하게 될 가능성을 배제할 수 없습니다.

따라서 이런 상황을 예방하고 최대한 제한하기 위해선 최소의 필요 값을 반환해야 합니다.

같은 이유로 엔터티를 그대로 반환하는 것 또한 지양하는 것이 좋습니다.

## 읽기 전용 유스케이스

읽기 전용 기능을 유스케이스라고 부르는 것은 조금 어색합니다. UI 레벨에서 '계좌 잔고 조회'라는 것 자체를 유스케이스로 특정할 수도 있겠지만 전체 프로젝트 맥락에서는 간단한 쿼리 작업이라고 할 수 있습니다. 그래서 유스케이스와 구분을 위해 쿼리로 표현할 수 있습니다.

책에서 제안하는 아키텍처 스타일에서 이를 구현하는 방법은 쿼리를 위한 인커밍 전용 포트를 만들고 쿼리 서비스에 구현하는 것입니다.

```java
package buckpal.application.service;

@RequiredArgsConstructor
class GetAccountBalanceService implements GetAccountBalanceQuery {

	private final LoadAccountPort loadAccountPort;

	@Override
	public Money getAccountBalance(AccountId accountId) {
		return loadAccountPort.loadAccount(accountId, LocalDateTime.now())
				.calculateBalance();
	}
}
```

쿼리 서비스 자체는 유스케이스 구현체인 서비스와 동일한 방식으로 동작합니다. 인커밍 포트를 통해 요청을 받아 아웃커밍 포트로 DB와 연동합니다.

이처럼 읽기 전용 쿼리는 코드상에서 명확하게 구분되는데 이런 방식은 CQS(Command-Query Separation)나 CQRS(Command-Query Responsibility Segregation)와 같은 개념과 잘 맞는다고 할 수 있습니다.

## 유지보수에 어떻게 도움이 될까?

도메인 로직은 원하는 대로 구현하면서 입출력 모델을 독립적으로 모델링해 원치 않는 부수효과를 피할 수 있게 됩니다.

반면 유스케이스 간 모델을 공유하는 것보다는 더 많은 작업이 필요합니다. 각 유스케이스마다 별도의 모델을 만들어 엔터티와 매핑해야 하기 때문입니다.

그러나 유스케이스별로 모델을 만들면 해당 유스케이스를 명확하게 이해할 수 있고 장기적으로도 유지보수에 도움이 됩니다.

그리고 각 유스케이스별로 서로 다른 사람이 동시에 작업을 하더라도 큰 문제가 발생하지 않습니다.