![](https://img.shields.io/badge/spring--boot-2.5.1-red) ![](https://img.shields.io/badge/gradle-7.0.2-brightgreen)

> 모든 소스 코드는 [여기](https://github.com/lcalmsky/spring-data-jpa)에서 확인 가능합니다.

## Hint

`SQL`에서 `Hint`를 사용하듯이 JPA에서도 `JPA 구현체`에 힌트를 전달할 수 있습니다.

### 조회만 사용하기(Read Only)

기본적으로 JPA를 이용해 데이터를 조회하게되면 영속성 컨텍스트에 저장되어 관리되고, 그 값을 수정한 뒤 `flush()`하거나 `dirty checking`이 발생하면 업데이트 쿼리도 발생하게 됩니다.

`Hint`를 사용하여 영속성 컨텍스트에 저장되는 것을 방지해 메모리 낭비를 막고 혹시 모를 변경사항에 대해 업데이트되지 않도록 할 수 있습니다.

먼저 `Repository`에 이름으로 사용자를 조회하는 메서드를 추가합니다.

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.entity.Member;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.QueryHints;
import org.springframework.stereotype.Repository;

import javax.persistence.QueryHint;

@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
    @QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true")) // (1)
    Member findMemberByUsername(String username);
}
```

> (1) `@QueryHints` 애너테이션의 속성으로 `@QueryHint`를 추가할 수 있습니다. JPA 구현체인 `hibernate`의 `readOnly` 옵션을 `true`로 지정합니다.

`@QueryHint` 애너테이션은 어떤 것도 다 전달받을 수 있게 하기위해 `String` 파라미터를 받고 있는데 사실 이 부분이 개인적으로는 매우 불편하다고 생각합니다.

뭔가 실수할 가능성이 높아지고 자동완성도 지원이 안 되고..

테스트 클래스에서 힌트가 적용되었는지 확인해봅시다.

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.entity.Member;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import javax.persistence.EntityManager;

@SpringBootTest
class MemberRepositoryTest {
    @Autowired
    MemberRepository memberRepository;
    @Autowired
    TeamRepository teamRepository;
    @Autowired
    EntityManager entityManager;

    @Test
    @Transactional // (1)
    void hint() {
        // given
        memberRepository.save(new Member("a", 10)); // (2)
        entityManager.flush();
        entityManager.clear();

        // when
        Member member = memberRepository.findMemberByUsername("a");
        member.changeUsername("b"); // (3)
        entityManager.flush(); // (4)
    }
}
```

(1) `flush()`를 사용할 것이기 때문에 `@Transactional` 애너테이션을 추가해줍니다.

(2) 사전 조건인 회원 한 명을 추가한 뒤 `flush()`, `clear()`를 호출해 영속성 컨텍스트에서 관리하지 않도록 합니다.

(3) 다시 `MemberRepository`에서 조회해 온 뒤 `Member` 객체의 값을 바꿔 `dirty checking`이 발생하는지 확인합니다.

(4) `flush()`를 호출해 해당 변경사항이 DB에 반영되는지 확인합니다.

`Hint`를 추가하지 않았을 때의 동작을 예상해보자면, (2)번에 의해 `insert` 쿼리가 한 번 발생하고, 다시 조회할 때 `select` 쿼리 한 번, 이름을 변경할 때 `update` 쿼리가 한 번 더 발생해야 합니다.

하지만 힌트를 사용한 결과는 아래와 같습니다.

```text
2021-07-05 07:18:19.611 DEBUG 37191 --- [           main] org.hibernate.SQL                        : 
    insert 
    into
        member
        (age, team_id, username, member_id) 
    values
        (?, ?, ?, ?)
2021-07-05 07:18:19.632 DEBUG 37191 --- [           main] org.hibernate.SQL                        : 
    select
        member0_.member_id as member_i1_0_,
        member0_.age as age2_0_,
        member0_.team_id as team_id4_0_,
        member0_.username as username3_0_ 
    from
        member member0_ 
    where
        member0_.username=?
```

사전 조건을 위한 `insert`문을 제외하면 `select` 쿼리 한 번만 발생했습니다.

이 말은 객체의 속성을 변경한 내용이 DB에 다시 반영되지 않는다는 말이고, 영속성 컨텍스트 관리를 위한 메모리를 추가로 사용하지 않았다는 말이기도 합니다.

이처럼 변경 사항이 DB에 반영될 필요 없이 오직 조회만 필요할 때 `readOnly` 옵션을 사용할 수 있습니다.

이 힌트를 이용해서는 극적인 효과를 보긴 어렵습니다만, 단순 조회인데 부하가 심한 API이거나 상당히 빈번하게 호출되는 경우 등에 한해 적용해보고, 성능테스트를 결과를 확인한 뒤 취할 수 있는 이점이 있는 경우에만 적용하시는 게 바람직합니다.

> `readOnly` 옵션 외에도 `comment`를 이용한 힌트 등도 제공하지만 이번 포스팅에서는 다루지 않습니다.

## Lock

`JPA`의 기본 동작이 `select` - `update` 이기 때문에 어떤 값을 동시에 여러 스레드에서 변경하려고 할 때 그 값의 `정합성`을 보장하기 어렵습니다.

`Lock`에는 여러 종류가 있지만 위의 상황에서 사용하는 비관적 잠금(Pessimistic Lock, 동일한 데이터를 동시에 수정할 가능성이 높다는 비관적인 전제로 잠금을 거는 방식)을 사용해야 합니다.

`JPA`에서 지원하는 `Lock` 기능을 `스프링 데이터 JPA`에서는 단순히 애너테이션만 추가해서 사용할 수 있습니다.

(`JPA`에서도 물론 파라미터 하나 추가정도로 간단히 사용 가능하지만 여기선 pass!)

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.entity.Member;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Lock;
import org.springframework.stereotype.Repository;

import javax.persistence.LockModeType;
import java.util.List;

@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Lock(LockModeType.PESSIMISTIC_WRITE) // (1)
    List<Member> findMembersByUsername(String username);
}
```

(1) `@Lock 애너테이션`의 속성을 이용해 비관적 잠금 모드를 사용하겠다고 명시하였습니다.

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.entity.Member;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import javax.persistence.EntityManager;
import javax.transaction.Transactional;
import java.util.List;

@SpringBootTest
class MemberRepositoryTest {
    @Autowired
    MemberRepository memberRepository;
    @Autowired
    TeamRepository teamRepository;
    @Autowired
    EntityManager entityManager;

    @Test
    @Transactional
    void lock() {
        // given
        memberRepository.save(new Member("a", 10));
        entityManager.flush();
        entityManager.clear();

        // when
        List<Member> member = memberRepository.findMembersByUsername("a");
    }
}
```

`@Lock` 애너테이션을 추가한 메서드를 테스트에서 호출해보면 결과는 아래와 같습니다.

```text
2021-07-05 07:44:22.562 DEBUG 38193 --- [           main] org.hibernate.SQL                        : 
    select
        member0_.member_id as member_i1_0_,
        member0_.age as age2_0_,
        member0_.team_id as team_id4_0_,
        member0_.username as username3_0_ 
    from
        member member0_ 
    where
        member0_.username=? for update
```

사전 조건인 `insert` 쿼리 로그는 제외하고 `select` 쿼리 로그인데 자세히 보시면 미세하게 다른 부분이 있습니다.

바로 마지막에 `for update`라는 구문이 추가된 것인데요, 이 구문이 있기 때문에 동시 다발적으로 업데이트가 될 때도 `정합성`이 보장됩니다. 

이 구문은 사용하는 `DB`마다 문법이 상이한데 `spring.jpa.properties.hibernate.dialect` 설정 값을 이용해 사용하는 `Dialect`를 지정하면 해당 `DB` 언어에 맞게 표현됩니다.

지원하는 `Lock` 모드는 굉장히 많지만 개인적으로는 위의 상황에서 사용할 일이 가장 많았던 거 같습니다.

자세한 내용은 `JPA`를 나중(?)에 다시 다룰 여유가 된다면 그 때 포스팅하려고 합니다. 