## 사용자 정의 Repository Best Practice

사용자 정의 `Repository`를 만들 때 가장 좋은 방법을 소개합니다.

일반적으로 사용자 정의 `Repository`라고 함은, `JpaRepository`를 상속해서 쿼리 메서드를 이용하는 것이 아니라 구현체에서 직접 `JPA`를 사용하거나 `MyBatis`, `JdbcTemplate`, `QueryDSL` 등을 이용해 구현한 것을 말합니다.

### 상대적으로(?) 나쁜 예시

`Entity`가 `Member`, `Team`이 존재하고 `Member`는 `JpaRepository`를 사용하고 `Team`은 사용자 정의 `Repository`를 사용한다고 가정하면 그냥 구현해도 상관없습니다. 하지만 `MemberRepository`가 존재하는데 `CustomMemberRepository`를 따로 만들어서 `Service` 레이어에 의존성을 따로 주입해서 사용한다고 가정했을 때 두 `Repository`가 뭐가 달라서 따로 구현되어있는지, 어떤 기능은 어디에 들어있는지 등을 협업하는 동료나 유지보수를 맡은 개발자가 직접 확인해야하는 불편함이 있습니다.

위의 예시를 다이어그램으로 표현해보면,

![custom-repository-bad-case-diagram](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/lcalmsky/spring-data-jpa/master/diagram/custom-repository-bad-case.plantuml)

이렇게 `MemberService`가 `MemberRepository`와 `CustomMemberRepository` 모두에 의존성을 가지는 관계로 표현이 됩니다.

이 방법이 꼭 나쁘다고만은 할 수 없습니다만, 위에서 언급한 것처럼 다소 번거롭다고 느낄 수 있고 이는 객체지향적으로도 제대로 된 설계라고 할 수 없을 뿐만 아니라 더 좋은 방법이 존재하기 때문에 굳이 나쁜 예시를 사용할 필요가 없습니다.

그렇다면 어떻게 바꿔야 할까요?

### Best Practice

다이어그램을 먼저 확인해봅시다.

![custom-repository-best-practice-diagram](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/lcalmsky/spring-data-jpa/master/diagram/custom-repository-best-practice.plantuml)

`MemberRepository`가 `CustomMemberRepository`를 상속하는 구조를 만들고 `CustomMemberRepositoryHandler`는 `CustomMemberRepository`를 구현하게 했습니다.

이런 구조가 된다면 `CustomMemberRepositoryHandler`는 `CustomMemberRepository`의 메서드만 `Override`하여 사용할 수 있습니다.

`MemberService`는 `MemberRepository`에만 의존관계를 갖습니다.

그렇다면 `@Autowired`나 생성자 주입을 통해 `MemberRepository`와의 의존성만을 주입했을 때 JPA가 직접 생성해주는 구현체와 사용자가 생성한 구현체를 구별해야 한다는 것인데, 다행스럽게도 갓프링님께서 직접 이런 번거로운 작업을 해주시니 우리는 감사한 마음으로 더 나은 객체지향적인 설계를 경험할 수 있습니다.

소스 코드로 확인해보겠습니다.

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.entity.Member;

public interface CustomMemberRepository {
    Member findByCustomFactor();
}
```

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.entity.Member;
import lombok.RequiredArgsConstructor;

import javax.persistence.EntityManager;

@RequiredArgsConstructor
public class CustomMemberRepositoryHandler implements CustomMemberRepository {
    private final EntityManager entityManager;

    @Override
    public Member findByCustomFactor() {
        return entityManager.createQuery("select m from Member m", Member.class).getSingleResult();
    }
}
```

먼저 `CustomMemberRepository` 인터페이스와 인터페이스를 구현하는 구현체인 `CustomMemberRepositoryHandler` 클래스를 작성합니다. 구현체에서는 `EntityManager`를 이용해 `JPA`를 사용하도록 하였는데 얼마든지 다른 프레임워크를 이용해 구현체를 직접 구현할 수 있고 쿼리도 복잡해질 수 있지만 귀찮아서 하나만 조회해오는 것으로 구현하였습니다. 예시는 예시로만 봐주세요 :sweat_smile: 

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.entity.Member;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface MemberRepository extends JpaRepository<Member, Long>, CustomMemberRepository { // (1)
    
}
```

> (1) `CustomMemberRepository`를 상속합니다.

잘 동작하는지 테스트 해봐야겠죠?

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.entity.Member;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import javax.persistence.EntityManager;
import javax.transaction.Transactional;

@SpringBootTest
class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;
    @Autowired
    EntityManager entityManager;

    @Test
    @Transactional
    @DisplayName("CustomRepository 기능 테스트")
    void customRepository() {
        // given
        memberRepository.save(new Member("a", 10));
        entityManager.flush();
        entityManager.clear();

        // when
        Member member = memberRepository.findByCustomFactor();
    }
}
```

한 번 실행해 볼까요?

```text
java.lang.IllegalStateException: Failed to load ApplicationContext
    ... 생략
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'memberRepository' defined in io.lcalmsky.springdatajpa.domain.repository.MemberRepository defined in @EnableJpaRepositories declared on JpaRepositoriesRegistrar.EnableJpaRepositoriesConfiguration: Invocation of init method failed; nested exception is org.springframework.data.repository.query.QueryCreationException: Could not create query for public abstract io.lcalmsky.springdatajpa.domain.entity.Member io.lcalmsky.springdatajpa.domain.repository.CustomMemberRepository.findByCustomFactor()! Reason: Failed to create query for method public abstract io.lcalmsky.springdatajpa.domain.entity.Member io.lcalmsky.springdatajpa.domain.repository.CustomMemberRepository.findByCustomFactor()! No property customFactor found for type Member!; nested exception is java.lang.IllegalArgumentException: Failed to create query for method public abstract io.lcalmsky.springdatajpa.domain.entity.Member io.lcalmsky.springdatajpa.domain.repository.CustomMemberRepository.findByCustomFactor()! No property customFactor found for type Member!
    ... 생략
Caused by: org.springframework.data.repository.query.QueryCreationException: Could not create query for public abstract io.lcalmsky.springdatajpa.domain.entity.Member io.lcalmsky.springdatajpa.domain.repository.CustomMemberRepository.findByCustomFactor()! Reason: Failed to create query for method public abstract io.lcalmsky.springdatajpa.domain.entity.Member io.lcalmsky.springdatajpa.domain.repository.CustomMemberRepository.findByCustomFactor()! No property customFactor found for type Member!; nested exception is java.lang.IllegalArgumentException: Failed to create query for method public abstract io.lcalmsky.springdatajpa.domain.entity.Member io.lcalmsky.springdatajpa.domain.repository.CustomMemberRepository.findByCustomFactor()! No property customFactor found for type Member!
```

이게 머선일인지 정상동작하지 않습니다.

로그 내용은 `Repository Bean` 생성에 실패해 `ApplicationContext`를 로드하지 못했다는 내용입니다.

왜 실패했는지 자세히 살펴보면 `CustomMemberRepository`의 `findByCustomFactor`에서 `customFactor`라는 `Member` 필드가 존재하지 않는다는 내용입니다.

이는 쿼리 메서드를 사용할 때 발생해야 하는 오류인데 분명 `Custom` 클래스를 구현했는데 왜 그런 걸까요?

다이어그램을 유심히 보신 분들은 눈치 채셨을 수도 있겠지만 다이어그램에는 구현체 클래스의 이름을 다소 전통적인 방식인 `Impl`을 붙여 구현하였고, 테스트 코드에는 `Handler`를 붙였습니다.

저같은 컨벤션충들에게는 다소 안 좋은 소식이지만 스프링에서 자동으로 구현체를 찾아주기위해 사용하는 `postfix`가 바로 `Impl` 입니다.

따라서 `CustomMemberRepositoryHandler`를 `CustomMemberRepositoryImpl`로 바꿔주면 정상동작하게됩니다.

IDE의 도움을 받아 한 번에 이름을 변경해주고 다시 테스트 해 봅시다.

```java
// 생략
public class CustomMemberRepositoryImpl implements CustomMemberRepository { // Handler -> Impl로 변경
    // 생략
}
```

```text
2021-07-07 09:28:32.780 DEBUG 40766 --- [           main] org.hibernate.SQL                        : 
    insert 
    into
        member
        (age, team_id, username, member_id) 
    values
        (?, ?, ?, ?)
2021-07-07 09:28:32.789 DEBUG 40766 --- [           main] org.hibernate.SQL                        : 
    select
        member0_.member_id as member_i1_0_,
        member0_.age as age2_0_,
        member0_.team_id as team_id4_0_,
        member0_.username as username3_0_ 
    from
        member member0_
```

`select m from Member m` 쿼리가 정상 동작한 것을 확인할 수 있습니다.

추가로 `Impl` 대신 다른 `postfix`를 사용할 수 있습니다.

`JPA` 설정(xml 파일)에 아래 부분을 추가하시면 됩니다.

```xml

<repositories xmlns="http://www.springframework.org/schema/data/jpa"
              base-package="io.lcalmsky.springdatajpa.domain.repository"
              repository-impl-postfix="Impl"/> <!-- (1) -->
```

> (1) `Repository` 구현체 이름의 `postfix`를 변경할 수 있습니다.

개인적으로는 스프링 데이터 `JPA`를 쓰면서 `JPA` XML 설정을 따로 작성하는 것을 매우 귀찮아 하는 데요, 스프링은 저 같은 미천한 개발자를 위해 `Java Configuration`도 제공합니다.

```java
@EnableJpaRepositories(basePackage = "io.lcalmsky.springdatajpa.domain.repository", repositoryImplementationPostfix = "Impl") // (1)
public class Foo {

}
```

> (1) `@EnableJpaRepositories` 애너테이션의 속성 중 `repositoryImplementationPostfix`를 이용해 변경 가능합니다.

위에 상대적으로 나쁜 예시라고 예를 들었으나 그 방법이 나쁘다고만은 볼 수 없습니다.

진리의 "상황에 따라", "케바케" 잊지 마세요!