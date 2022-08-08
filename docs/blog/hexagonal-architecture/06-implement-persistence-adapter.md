> 이 포스팅은 [만들면서 배우는 클린 아키텍처](http://www.yes24.com/Product/Goods/105138479)를 읽고 작성하였습니다.  
> 소스 코드는 [여기](https://github.com/thombergs/buckpal.git) 있습니다.

## Overview

처음 계층적 아키텍처의 문제접을 다룰 때 모든 것이 영속성 계층에 의존하게 되어 데이터베이스 주도 설계가 된다는 점을 다룬 적이 있는데요, 의존성을 역전시키기 위해 영속성 계층을 애플리케이션 계층의 플러그인으로 만드는 방법을 살펴봅니다.

## 의존성 역전

![](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/lcalmsky/lcalmsky/master/docs/blog/hexagonal-architecture/06-01.puml)

애플리케이션 서비스에서는 영속성을 사용하기위해 아웃고잉 포트 인터페이스를 호출합니다. 이 포트는 영속성 작업을 수행하고 데이터베이스와 통신할 영속성 어댑터가 구현하게 됩니다.

헥사고날 아키텍처에서 영속성 어댑터는 아웃고잉 어댑터에 해당하므로 애플리케이션 계층에서 호출하고 역방향으로 어댑터가 애플리케이션을 호출할 일이 없습니다.

애플리케이션 서비스가 영속성을 신경쓰지 않고 도메인 코드를 개발하기 위해 영속성 사이의 인터페이스인 포트를 이용합니다. 포트가 존재하기 떄문에 영속성을 리팩터링 하더라도 애플리케이션 코드를 변경할 일이 없습니다.

## 영속성 어댑터의 책임

영속성 어댑터의 책임은 다음과 같습니다.

1. 입력을 받음
2. 입력을 데이터베이스 포맷으로 매핑
3. 입력을 데이터베이스로 전송
4. 데이터베이스 출력을 애플리케이션 포맷으로 매핑
5. 출력을 반환

영속성 어댑터는 포트를 통해 입력 받는데, 입력 모델은 인터페이스가 지정한 도메인 엔터티 또는 데이터베이스 연산을 위한 객체 입니다.

그리고 데이터베이스에 쿼리하기 위해 알맞은 포맷으로 입력 모델을 매핑합니다. JPA를 사용하는 경우 JPA 엔터티 객체로 매핑하는데 수많은 매핑이 번거로울 수 있어서 매핑하지 않는 전략도 사용할 수 있습니다. 이 부분은 이후 경계 간 매핑에서 다루도록 하겠습니다.

JPA 대신 다른 프레임워크를 사용하더라도 전혀 상관 없고 입력 모델이 꼭 객체가 아니라 파일이어도 상관 없습니다. 핵심은 영속성 어댑터의 입력 모델이 영속성 어댑터 패키지와 같이 있는 것이 아니라 애플리케이션 코어에 있기 때문에 어댑터의 내용을 변경하는 것이 애플리케이션에 영향을 미치지 않습니다.

다음으로 데이터베이스와 통신하여 쿼리하고 결과를 받아오게 됩니다. 받아온 결과를 포트에 정의된 출력 모델로 매핑하여 반환하면 이 또한 애플리케이션 코어에 위차한 모델로 매핑하게 됩니다.

## 포트 인터페이스 나누기

서비스를 구현하다보면 데이터베이스와 연동하는 포트 인터페이스를 어떻게 나눠야할지 의문이 생기게 됩니다. 보통 특정 엔터티가 필요로하는 모든 데이터베이스 연산을 하나의 리파지토리 인터페이스에 두는 것이 일반적인 방법입니다.

![](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/lcalmsky/lcalmsky/master/docs/blog/hexagonal-architecture/06-02.puml)

이렇게하면 데이터베이스 연산에 의존하는 각 서비스는 포트 인터페이스에 넓은 의존성을 가지게 됩니다. 즉, 더 작게 나눌 수 있게되므로 이는 불필요한 의존성이 생길 수 있다는 뜻입니다.

불필요한 메서드에 생긴 의존성은 코드를 이해하고 테스트하기 어렵게 만듭니다. RegisterAccountService의 단위테스트를 작성한다고 가정하면 AccountRepository 인터페이스의 어떤 메서드를 모킹해야할지 내용을 다 알고있어야 합니다. 내용을 알아서 인터페이스의 일부만 모킹하다고 해도 다른 사람이 테스트할 때 전체가 모킹되었다고 기대하는 경우 에러가 발생할 수 있습니다.

인터페이스 분리 원칙(Interface Segregation Principle, ISP)을 이용해 이 문제를 해결할 수 있습니다. 이 원칙은 클라이언트가 오직 자신이 필요로 하는 메서드만 알 수 있도록 인터페이스를 분리하는 것입니다.

![](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/lcalmsky/lcalmsky/master/docs/blog/hexagonal-architecture/06-03.puml)

이렇게 인터페이스를 분리하면 각 서비스는 실제 필요한 메서드에만 의존하게 됩니다. 그리고 포트 이름이 포트의 역할을 잘 표현하고 있어 테스트에서 어떤 포트와 메서드를 모킹할지 고민할 필요가 없습니다.

이렇게 매우 좁은 포트를 만들게 되면 코딩 자체를 플러그 앤 플레이 경험으로 만들어줍니다. 필요한 것만 꽂아서 사용하면 되는 것이죠.

모든 상황에 대해서 포트당 하나의 메서드를 적용하는 것이 바람직하진 않을 것입니다. 경우에 따라 응집성이 높은 인터페이스가 필요할 수 있기 때문입니다.

## 영속성 어댑터 나누기

이전까지는 영속성 어댑터가 여러 포트를 구현하는 하나의 클래스였습니다. 하지만 꼭 하나로 만들 필요 없이 영속성 연동이 필요한 도메인 클래스 하나당 하나의 어댑터를 구현하는 방식을 택할 수도 있습니다.

![](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/lcalmsky/lcalmsky/master/docs/blog/hexagonal-architecture/06-04.puml)

이렇게 구현하면 영속성 어댑터들은 각 도메인을 따라 자동으로 나눠지게 됩니다.

어댑터를 더 많은 종류로도 나눌 수 있습니다. JPA를 사용하는 어댑터, Querydsl을 사용하는 어댑터, JdbcTemplate을 사용하는 어댑터 등을 사용해 아웃고잉 포트의 인터페이스를 구현할 수 있습니다.

도메인 코드는 영속성 포트의 명세만 사용하기 때문에 뒷단의 구현 내용을 전혀 알 필요가 없습니다.

## 스프링 데이터 JPA 예제

데이터베이스와 통신하기 위해 JPA를 사용할 것이므로 계좌의 엔터티 클래스가 필요합니다.

현재 단계에서는 계좌의 ID만 사용하고 있는데 나중에 사용자 ID 등의 필드가 추가되어야 합니다.

```java
package buckpal.adapter.persistence;

@Entity
@Table(name = "account")
@Data
@AllArgsConstructor
@NoArgsConstructor
class AccountJpaEntity {

	@Id
	@GeneratedValue
	private Long id;

}
```

다음은 특정 계좌의 모든 활동을 저장하는 테이블을 표현하기 위한 엔터티 클래스 입니다.

```java
package buckpal.adapter.persistence;

@Entity
@Table(name = "activity")
@Data
@AllArgsConstructor
@NoArgsConstructor
class ActivityJpaEntity {

	@Id
	@GeneratedValue
	private Long id;

	@Column
	private LocalDateTime timestamp;

	@Column
	private Long ownerAccountId;

	@Column
	private Long sourceAccountId;

	@Column
	private Long targetAccountId;

	@Column
	private Long amount;

}
```

다음으로는 계좌와 계좌 활동에 CRUD 작업을 하기위한 리파지토리 인터페이스 입니다.

```java
package buckpal.adapter.out.persistence;

interface AccountRepository extends JpaRepository<AccountJpaEntity, Long> {
}
```

```java
package buckpal.adapter.out.persistence;

interface ActivityRepository extends JpaRepository<ActivityJpaEntity, Long> {

	@Query("select a from ActivityJpaEntity a " +
			"where a.ownerAccountId = :ownerAccountId " +
			"and a.timestamp >= :since")
	List<ActivityJpaEntity> findByOwnerSince(
			@Param("ownerAccountId") Long ownerAccountId,
			@Param("since") LocalDateTime since);

	@Query("select sum(a.amount) from ActivityJpaEntity a " +
			"where a.targetAccountId = :accountId " +
			"and a.ownerAccountId = :accountId " +
			"and a.timestamp < :until")
	Long getDepositBalanceUntil(
			@Param("accountId") Long accountId,
			@Param("until") LocalDateTime until);

	@Query("select sum(a.amount) from ActivityJpaEntity a " +
			"where a.sourceAccountId = :accountId " +
			"and a.ownerAccountId = :accountId " +
			"and a.timestamp < :until")
	Long getWithdrawalBalanceUntil(
			@Param("accountId") Long accountId,
			@Param("until") LocalDateTime until);
}
```

ActivityRepository에는 커스텀 쿼리를 사용합니다.

다음으로 영속성 기능을 제공하기 위한 영속성 어댑터를 구현합니다.

먼저 영속성 어댑터는 아웃고잉 포트를 구현하는데 해당 포트는 다음과 같습니다.

```java
package buckpal.application.port.out;

public interface LoadAccountPort {

	Account loadAccount(AccountId accountId, LocalDateTime baselineDate);
}
```

```java
package buckpal.application.port.out;

public interface UpdateAccountStatePort {

	void updateActivities(Account account);

}
```

이 두 포트를 구현한 어댑터는 다음과 같습니다.

```java
package buckpal.adapter.out.persistence;

@RequiredArgsConstructor
@Component
class AccountPersistenceAdapter implements
		LoadAccountPort,
		UpdateAccountStatePort {

	private final SpringDataAccountRepository accountRepository;
	private final ActivityRepository activityRepository;
	private final AccountMapper accountMapper;

	@Override
	public Account loadAccount(
					AccountId accountId,
					LocalDateTime baselineDate) {

		AccountJpaEntity account =
				accountRepository.findById(accountId.getValue())
						.orElseThrow(EntityNotFoundException::new);

		List<ActivityJpaEntity> activities =
				activityRepository.findByOwnerSince(
						accountId.getValue(),
						baselineDate);

		Long withdrawalBalance = orZero(activityRepository
				.getWithdrawalBalanceUntil(
						accountId.getValue(),
						baselineDate));

		Long depositBalance = orZero(activityRepository
				.getDepositBalanceUntil(
						accountId.getValue(),
						baselineDate));

		return accountMapper.mapToDomainEntity(
				account,
				activities,
				withdrawalBalance,
				depositBalance);

	}

	private Long orZero(Long value){
		return value == null ? 0L : value;
	}


	@Override
	public void updateActivities(Account account) {
		for (Activity activity : account.getActivityWindow().getActivities()) {
			if (activity.getId() == null) {
				activityRepository.save(accountMapper.mapToJpaEntity(activity));
			}
		}
	}

}

```

코드 자체는 우리가 그동안 서비스 레이어에서 리파지토리를 사용해 수행했던 것과 같습니다. 한 가지 다른 점은 Account, Activity 도메인 모델과 AccountJpaEntity, ActivityJpaEntity 데이터베이스 모델 간 양방향 매핑이 존재한다는 점입니다. 굳이 왜 이런 귀찮은 작업을 해야하는 걸까요?

앞서도 언급한 적 있지만 계층 간 매핑에 대해서는 추후 다루게 될 것인데, 매핑하지 않는 방법도 존재하긴 합니다. 단지 JPA 때문에 도메인 모델과 타협을 해야하는 단점이 있습니다. JPA 사용을 위해 반드시 필요한 기본생성자를 생성하거나 엔터티간 관계를 매핑해 주는 부분이 이에 해당합니다.

이런 부분을 신경쓰지 않고 풍부한 도메인 모델을 생성하기 위해서는 도메인 모델과 영속성 모델을 따로 매핑해주는 과정이 필요합니다.

## 데이터베이스 트랜잭션

트랜잭션의 경계는 어디 위치하는 것이 좋을까요?

트랜잭션은 하나의 유스케이스에 대해 일어나는 모든 쓰기 작업에 걸쳐 있어야 하나라도 실패했을 때 롤백될 수 있습니다.

영속성 어댑터는 어떤 기능이 어떤 유스케이스에 포함되는지 알지 못하기 때문에 트랜잭션을 직접 컨트롤하면 안 됩니다. 이 책임은 아웃고잉 포트를 통해 영속성 어댑터를 호출하는 주체인 서비스에 위임해야 합니다.

스프링에서 트랜잭션을 관리하기 가장 쉬운 방법은 `@Transactional` 애너테이션을 사용하는 것입니다. 서비스 클래스에 직접 사용하여 모든 public 메서드에 대해 동작할 수 있게 합니다.

```java
@Transactional
public class SendMoneyService implements SendMoneyUseCase {
  // 생략
}
```

## 유지보수에 어떻게 도움이 될까?

도메인 코드에 플러그인처럼 동작하는 영속성 어댑터를 이용해 도메인 코드와 완전히 분리되어 풍부한 도메인 모델을 만들 수 있습니다.

좁은 포트 인터페이스를 사용해 포트마다 다르게 구현할 수 있는 유연성을 확보할 수 있고 영속성 프레임워크나 기술 또한 포트의 명세를 지키는 선에서 얼마든지 교체할 수 있습니다.