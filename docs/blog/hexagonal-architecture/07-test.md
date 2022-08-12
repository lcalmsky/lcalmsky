> 이 포스팅은 [만들면서 배우는 클린 아키텍처](http://www.yes24.com/Product/Goods/105138479)를 읽고 작성하였습니다.  
> 소스 코드는 [여기](https://github.com/thombergs/buckpal.git) 있습니다.

## Overview

헥사고날 아키텍처에서의 테스트 전략에 대해 살펴봅니다.

## 테스트 피라미드

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/master/docs/blog/hexagonal-architecture/test-pyramid.png)

위 그림은 몇 개의, 어떤 종류의 테스트를 목표로 해야 하는지 결정하는 데 도움을 줍니다.

기본적으로 만드는 비용이 적고, 유지보수하기 쉽고, 빨리 실행되고, 안정적인 작은 크기의 테스트들에 대해 높은 커버리지를 유지해야 합니다.

여러 개의 단위와 경계를 결합하는 테스트는 비용이 비싸지고 실행이 느려져 깨지기 쉬워집니다. 테스트 피라미드는 테스트가 비싸질 수록 커버리지를 낮게 잡아야 한다는 것을 보여줍니다. 그렇지 않으면 기능구현보다 테스트 구현에 시간이 더 오래 걸리는 배보다 배꼽이 더 큰 상황이 발생하게 됩니다.

단위 테스트는 피라미드의 기반이 됩니다. 일반적으로 하나의 클래스를 인스턴스화 하고, 해당 클래스의 인터페이스를 통해 기능들을 테스트 합니다. 만약 테스트하는 클래스가 다른 클래스를 의존한다면 의존되는 클래스들은 인스턴스화하지 않고 목(mock)으로 대체합니다.

다음 계층은 통합테스트로 연결된 여러 유닛을 인스턴스화하고 시작점이 되는 클래스의 인터페이스를 호출하여 기대한대로 잘 동작하는지 확인합니다. 헥사고날 아키텍처에서의 통합테스트는 두 계층간의 경계를 걸쳐서 테스트할 수 있기 때문에 목 작업이 필요할 수 있습니다.

시스템 테스트는 특정 유스케이스에 대해 전 계층에서 잘 동작하는지 검증합니다.

시스템 테스트 상위에는 UI를 포함하는 테스트가 있을 수 있습니다.

지금까지 테스트 피라미드를 구성할 테스트를 정하고, 각 테스트에 대해 정의했습니다. 헥사고날 아키텍처의 각 계층에는 어떤 테스트가 가장 적합한지 살펴보겠습니다.

## 단위 테스트: 도메인 엔터티

헥사고날 아키텍처의 중심을 차지하고 있는 도메인 엔터티를 살펴보겠습니다.

이전 포스팅에서 다루었던 `Account` 엔터티의 테스트를 만들고 검증해보겠습니다.

```java
class AccountTest {

  @Test
  void withdrawalSucceeds() {
    AccountId accountId = new AccountId(1L);
    Account account = defaultAccount()
        .withAccountId(accountId)
        .withBaselineBalance(Money.of(555L))
        .withActivityWindow(new ActivityWindow(
            defaultActivity()
                .withTargetAccount(accountId)
                .withMoney(Money.of(999L)).build(),
            defaultActivity()
                .withTargetAccount(accountId)
                .withMoney(Money.of(1L)).build()))
        .build();

    boolean success = account.withdraw(Money.of(555L), new AccountId(99L));

    assertThat(success).isTrue();
    assertThat(account.getActivityWindow().getActivities()).hasSize(3);
    assertThat(account.calculateBalance()).isEqualTo(Money.of(1000L));
  }
}
```

`Account` 객체를 생성하고 `withdraw` 메서드를 호출하여 출금이 성공했는지 검증하고, `Account` 객체의 상태에 대해 확인하는 단순한 테스트 입니다.

이 테스트는 만들고 이해하는 게 쉽고, 아주 빠르게 실행됩니다. 이런 식의 단위 테스트야말로 도메인 엔터티의 비즈니스 규칙을 검증하기 가장 적절한 방법입니다. 도메인 엔터티는 다른 클래스에 거의 의존하지 않기 떄문에 다른 테스트는 필요하지 않습니다.

## 단위 테스트: 유스케이스

다음으로 유스케이스를 테스트합니다. 마찬가지로 이전 포스팅에서 다뤘던 `SendMoneyService`의 테스트를 살펴 보겠습니다. `SendMoney` 유스케이스는 출금 계좌의 잔고가 다른 트랜잭션에 의해 변경되지 않도록 락을 걸고 출금 계좌에서 돈이 출금되고나면 입금계좌에 락을 걸고 입금시킵니다. 그리고 두 계좌 모두 락을 해제합니다.

```java
class SendMoneyServiceTest {

  // 생략
  @Test
  void transactionSucceeds() {

    Account sourceAccount = givenSourceAccount();
    Account targetAccount = givenTargetAccount();

    givenWithdrawalWillSucceed(sourceAccount);
    givenDepositWillSucceed(targetAccount);

    Money money = Money.of(500L);

    SendMoneyCommand command = new SendMoneyCommand(
        sourceAccount.getId().get(),
        targetAccount.getId().get(),
        money);

    boolean success = sendMoneyService.sendMoney(command);

    assertThat(success).isTrue();

    AccountId sourceAccountId = sourceAccount.getId().get();
    AccountId targetAccountId = targetAccount.getId().get();

    then(accountLock).should().lockAccount(eq(sourceAccountId));
    then(sourceAccount).should().withdraw(eq(money), eq(targetAccountId));
    then(accountLock).should().releaseAccount(eq(sourceAccountId));

    then(accountLock).should().lockAccount(eq(targetAccountId));
    then(targetAccount).should().deposit(eq(money), eq(sourceAccountId));
    then(accountLock).should().releaseAccount(eq(targetAccountId));

    thenAccountsHaveBeenUpdated(sourceAccountId, targetAccountId);
  }
  // 생략
}
```

출금 및 입금하기 위핸 `Account` 객체를 각각 생성 및 초기화하고 `SendMoneyCommand` 객체를 생성해 유스케이스의 입력 모델로 사용하였습니다. 유스케이스 실행을 위해 `sendMoney` 메서드를 호출하였고 트랜잭션 성공 여부와 락을 걸고 해제하는 메서드가 호출되었는지 검증하였습니다.

코드에는 생략되어있지만 `given`으로 시작하는 메서드는 모킹 작업을 하고 `then` 메서드로 특정 메서드가 호출되었는지 확인합니다.

테스트 중인 유스케이스는 상태가 없기 때문에 `then`을 이용해 상태를 검증할 수 없습니다. 대신 모킹된 의존 대상과 상호작용했는지 여부를 검증합니다. 이는 코드의 행동 뿐 아니라 구조 변경에도 취약해진다는 의미로 코드가 리팩터링 된다면 테스트도 변경될 확률이 올라갑니다.

따라서 테스트에서 어떤 것을 검증하고 싶은지 잘 생각해봐야 합니다. 이전 단위테스트 처럼 모든 동작을 검증하는 것이 아닌, 중요한 핵심만 골라 집중해서 테스트해야 합니다. 만약 모든 동작을 검증하려고하면 테스트 비용이 늘어나고 이는 테스트 자체의 가치를 떨어트리는 일이 됩니다.

이 테스트는 단위테스트 이지만 의존성이 있는 객체와의 상호작용을 검증하기 때문에 통합테스트에 가깝습니다. 하지만 실제 의존성을 관리하는 대신 목을 이용하기 때문에 통합테스트에 비해 만들기 쉽고 유지보수하기 쉽습니다.

## 통합 테스트: 웹 어댑터

웹 어댑터는 HTTP 요청을 입력받아 유효성 검증을 하고 유스케이스 입력 모델로 매핑하고 전달합니다. 그리고 나서 유스케이스의 결과물을 다시 응답 객체로 매핑하여 클라이언트에게 반환합니다. 이 모든 단계가 제대로 동작하는지 검증이 필요합니다.

```java

@WebMvcTest(controllers = SendMoneyController.class)
class SendMoneyControllerTest {

  @Autowired
  private MockMvc mockMvc;

  @MockBean
  private SendMoneyUseCase sendMoneyUseCase;

  @Test
  void testSendMoney() throws Exception {

    mockMvc.perform(post("/accounts/send/{sourceAccountId}/{targetAccountId}/{amount}",
            41L, 42L, 500)
            .header("Content-Type", "application/json"))
        .andExpect(status().isOk());

    then(sendMoneyUseCase).should()
        .sendMoney(eq(new SendMoneyCommand(
            new AccountId(41L),
            new AccountId(42L),
            Money.of(500L))));
  }
}
```

스프링 부트에서 Controller를 테스트하기위한 방법과 동일한 방법입니다. 

웹 어댑터의 책임 대부분이 이 테스트로 커버되는데 MockMvc 객체가 이미 모킹된 객체이기 떄문에 실제로 HTTP를 이용한 테스트가 이뤄졌다고 볼 순 없습니다. 하지만 프레임워크를 믿고 적절히 변환되었다고 믿고 넘어갈 수 있습니다.

하지만 HTTP 요청을 SendMoneyCommand로 매핑하는 과정은 직접 다루고 있습니다. 만약 이 부분도 검증하는 커맨드로 생성했다면 유스케이스 모델로 매핑하는 부분에 대한 검증도 필요해 질 것입니다.

웹 컨트롤러 자체가 스프링 프레임워크와 강력하게 묶여있기 때문에 프레임워크와 통합된 상태로 테스트하는 것이 합리적이므로 프레임워크가 담당하는 부분은 그냥 믿고 맡기는 것이 프레임워크가 담당하는 부분을 직접 테스트하는 것보다 훨씬 나은 방법입니다.

## 통합 테스트: 영속성 어댑터

영속성 어댑터 테스트 역시 마찬가지로 프레임워크에 많이 의존하고 있고, 계층간 검증이 필요하므로 단위 테스트보다 통합 테스트를 적용하는 것이 합리적입니다. 단순한 어댑터 로직 뿐만 아니라 DB 매핑까지 검증해야 하기 때문입니다.

```java
@DataJpaTest
@Import({AccountPersistenceAdapter.class, AccountMapper.class})
class AccountPersistenceAdapterTest {

	@Autowired
	private AccountPersistenceAdapter adapterUnderTest;

	@Autowired
	private ActivityRepository activityRepository;

	@Test
	@Sql("AccountPersistenceAdapterTest.sql")
	void loadsAccount() {
		Account account = adapterUnderTest.loadAccount(new AccountId(1L), LocalDateTime.of(2018, 8, 10, 0, 0));

		assertThat(account.getActivityWindow().getActivities()).hasSize(2);
		assertThat(account.calculateBalance()).isEqualTo(Money.of(500));
	}

	@Test
	void updatesActivities() {
		Account account = defaultAccount()
				.withBaselineBalance(Money.of(555L))
				.withActivityWindow(new ActivityWindow(
						defaultActivity()
								.withId(null)
								.withMoney(Money.of(1L)).build()))
				.build();

		adapterUnderTest.updateActivities(account);

		assertThat(activityRepository.count()).isEqualTo(1);

		ActivityJpaEntity savedActivity = activityRepository.findAll().get(0);
		assertThat(savedActivity.getAmount()).isEqualTo(1L);
	}

}
```

앞서 구현한 영속성 어댑터는 `Account` 엔터티를 조회하는 기능과 계좌 활동을 DB에 저장하는 기능, 총 두 개의 기능으로 구성되어 있었습니다.

`@DataJpaTest` 애너테이션을 이용해 스프링 데이터 리파지토리를 포함하여 DB 접근에 필요한 객체를 인스턴스화 할 수 있습니다.

조회 메서드에 대해서는 SQL 스크립트를 이용해 미리 DB에 데이터를 저장해놓고 SQL로 업데이트한 상태와 동일한지 검증합니다.

계좌 활동을 저장하는 메서드에 대한 테스트는 반대로 동작합니다. 새로운 계좌 활동을 가진 `Account` 객체를 만들어 어댑터로 전달하고, 그 이후에 리파지토리 API를 이용해 잘 저장되었는지 확인합니다.

이 테스트에서는 DB를 모킹하지 않았기 떄문에 실제로 DB에 모킹합니다. DB를 모킹하더라도 마찬가지로 높은 코드 커버리지를 보였겠지만 SQL 구문 오류나 매핑 에러를 확인할 수 없으므로 DB의 경우 모킹하는 대신 메모리 기반 DB 등을 대신 사용할 수 있습니다.

하지만 프로덕션 환경에서는 메모리 기반 DB를 사용하지 않으므로 테스트가 잘 통과했더라도 실제 DB에서는 문제가 생길 가능성이 있습니다. 사용하는 DB의 종류가 다르면 문법이 다를 수 있기 때문입니다.

이러한 이유로 영속성 어댑터 테스트는 실제 DB를 대상으로 진행해야 합니다. 특정 라이브러리는 DB를 도커 컨테이너에 띄울 수 있게 해주므로 이럴 때 매우 유용하게 사용할 수 있습니다.

개인적인 생각으로는 DB 트랜잭션이 모두 `JPA`나 `Querydsl` 등 `EntityManager`를 통해 이뤄지는 경우라면 크게 신경쓸 필요가 없을 거 같습니다. DB 고유의 기능을 따로 사용하지 않는 이상 `dialect`를 이용해 알아서 설정한 드라이버에 맞게 변경해주기 때문입니다.

## 시스템 테스트: 주요 경로 테스트

피라미드 최상단에 위치한 시스템 테스트는 전체 애플리케이션을 띄우고 API 요청부터 모든 계층이 조화롭게 동작하는지 검증합니다.

송금하기 유스케이스에 대해 시스템 테스트를 작성한 것은 아래와 같습니다.

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class SendMoneySystemTest {

  @Autowired
  private TestRestTemplate restTemplate;

  // 생략

  @Test
  @Sql("SendMoneySystemTest.sql")
  void sendMoney() {

    Money initialSourceBalance = sourceAccount().calculateBalance();
    Money initialTargetBalance = targetAccount().calculateBalance();

    ResponseEntity response = whenSendMoney(
        sourceAccountId(),
        targetAccountId(),
        transferredAmount());

    then(response.getStatusCode())
        .isEqualTo(HttpStatus.OK);

    then(sourceAccount().calculateBalance())
        .isEqualTo(initialSourceBalance.minus(transferredAmount()));

    then(targetAccount().calculateBalance())
        .isEqualTo(initialTargetBalance.plus(transferredAmount()));

  }
  // 생략
  private ResponseEntity whenSendMoney(
      AccountId sourceAccountId,
      AccountId targetAccountId,
      Money amount) {
    HttpHeaders headers = new HttpHeaders();
    headers.add("Content-Type", "application/json");
    HttpEntity<Void> request = new HttpEntity<>(null, headers);

    return restTemplate.exchange(
        "/accounts/send/{sourceAccountId}/{targetAccountId}/{amount}",
        HttpMethod.POST,
        request,
        Object.class,
        sourceAccountId.getValue(),
        targetAccountId.getValue(),
        amount.getAmount());
  }
  // 생략
}

```

`@SpringBootTest` 애너테이션을 이용해 스프링 구동에 필요한 컴포넌트들을 구성하게 하고 랜덤 포트로 애플리케이션을 띄울 수 있게 합니다.

테스트 메서드에서는 요청을 생성해서 애플리케이션에 보내고 응답 상태와 계좌의 새로운 잔고를 확인합니다.

여기서는 웹 어댑터처럼 `MockMvc`를 이용하는 것이 아니라 `RestTemplate`을 이용해 로컬 서버와 통신하여 실제 `HTTP` 통신을 합니다.

아웃고잉 어댑터도 모킹하지 않고 빈에 등록된 컴포넌트를 이용합니다. 다른 시스템과 연동테스트가 필요한 경우에는 모킹을 해야 할 때도 있습니다. 헥사고날 아키텍처는 이런 경우 출력 포트 인터페이스만 모킹하면 되기 때문에 아주 쉽게 테스트를 이어나갈 수 있습니다.

단위 테스트와 통합 테스트를 만들었다면 시스템 테스트는 이미 만든 테스트에서 커버하는 범위와 겹치는 부분이 많을텐데 왜 해야 할까요? 일반적으로 시스템 테스트는 단위 테스트, 통합 테스트를 통해 발견할 수 있는 버그와는 다른 종류의 버그를 발견할 수 있게 해줍니다. 계층 간 매핑 버그가 이에 해당합니다. 게다가 여러 개의 유스케이스를 결합한 시나리오가 있을 경우 다른 테스트로는 진행할 수 없습니다. 시스템 테스트를 통해 코드 커버리지가 아닌 시나리오 자체를 커버할 수 있다면 배포될 준비가 되었다는 확신을 가질 수 있습니다.

개인적으로는 시나리오 테스트라는 이름을 사용하고 몇 개의 API를 호출해야하는 시나리오가 존재하는 경우에 적합한 테스트라고 생각합니다.

## 얼만큼 테스트해야 하나

테스트의 척도로 많이 사용하는 기준 중 코드 커버리지가 있습니다. 구현된 내용이 필요한 코드를 모두 포함하는지를 테스트하는 것이죠. 하지만 이런 커버리지는 테스트 성공을 판단하기에 정확한 지표로 사용하긴 어렵습니다. 그냥 팀이나 같이 일하는 동료들끼리의 약속 정도로 더 나은 코드를 작성할 수 있게 해주는 강제성을 높이기 위한 용도가 더 크다고 이야기 할 수 있죠.

커버리지가 100%라고 해도 일부 시나리오에서는 제대로 동작하지 않을 수도 있습니다. 따라서 테스트의 기준은 '마음 편하게 배포할 수 있는가?'가 되어야 합니다. 테스트가 배포를 해도 될 만큼 신뢰되는 수준으로 작성해야 합니다.

배포 이후 버그에 대해서는 수정하면서 배우는 내용을 꼭 기록해놓고 그 케이스도 커버할 수 있는 테스트를 추가해두어야 합니다.

그래도 테스트에 대한 기준은 있어야겠죠. 다음은 헥사고날 아키텍처에서 테스트를 위해 사용하는 전략입니다.

* 도메인 엔터티를 구현할 때는 단위 테스트로 커버
* 유스케이스를 구현할 때는 단위 테스트로 커버
* 어댑터를 구현할 때는 통합 테스트로 커버
* 시나리오는 시스템 테스트로 커버

## 유지보수에 어떻게 도움이 될까?

헥사고날 아키텍처는 도메인 로직과 바깥 방향으로 향하는 어댑터를 깔끔하게 분리하므로 도메인 로직은 단위 테스트로, 어댑터는 통합 테스트로 처리하는 전략을 세울 수 있습니다.

입출력 포트만 모킹하거나 실제로 구현하면 되고, 포트가 제공하는 인터페이스가 좁을 수록 모킹 범위도 좁아져 테스트에 유리합니다.

모킹이 버거워지거나 특정 부분을 커버하기 위해 어떤 테스트를 사용해야할지 모르겠다면 이는 특정 계층이 비대해지고 있다는 경고 신호입니다.