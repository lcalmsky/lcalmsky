![](https://img.shields.io/badge/spring--boot-2.5.1-red) ![](https://img.shields.io/badge/gradle-7.0.2-brightgreen)

> 모든 소스 코드는 [여기](https://github.com/lcalmsky/spring-data-jpa)에서 확인 가능합니다.

JPA는 보통 데이터를 가져와서 변경하면 변경 감지(dirty checking)를 통해 DB에 업데이트 퀴리를 수행합니다.

이런 업데이트들은 건 별로 `select` 이후 `update`가 이루어지기 때문에 수천 건을 업데이트 해야하는 경우 비효율적일 수 있습니다.

JPA를 사용해서도 수 천, 수 만 건의 데이터를 한 번에 업데이트 하는 벌크 업데이트(Bulk Update)쿼리를 사용할 수 있습니다.

### @Modifying 애너테이션

벌크 업데이트를 하기 위해선 `@Query`와 함께 `@Modifying` 애너테이션을 사용해야 합니다.

나이가 N살 이상인 전체 회원의 나이를 1씩 증가시켜야 한다는 요구사항이 존재한다고 가정하고 이를 구현한 소스 코드 입니다.

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.entity.Member;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Modifying
    @Query("update Member m set m.age = m.age + 1 where m.age >= :age")
    int increaseAgeOfAllMembersOver(@Param("age") int age);
}
```

이런식으로 전체 업데이트 쿼리를 작성할 수 있습니다.

이 때 @Modifying 애너테이션을 누락시키면 아래와 같은 에러가 발생합니다.

```text
org.springframework.dao.InvalidDataAccessApiUsageException: org.hibernate.hql.internal.QueryExecutionRequestException: Not supported for DML operations 
```

DML operation을 지원하지 않는다는 내용인데요, JPA 기본 동작이 `select` - `update`로 이루어져있기 때문에 바로 `update`만 하겠다고 알려주는 애너테이션이 바로 `@Modifying` 애너테이션 입니다.

`@Modifying` 애너테이션을 사용할 때 주의할 점은 반환 타입을 반드시 `void`나 `int` 또는 `Integer`로 지정해야 한다는 것 입니다.

만약 `long`같은 다른 타입으로 지정하게 되면 다음과 같은 에러가 발생합니다.

```text
org.springframework.dao.InvalidDataAccessApiUsageException: Modifying queries can only use void or int/Integer as return type! Offending method: public abstract long io.lcalmsky.springdatajpa.domain.repository.MemberRepository.increaseAgeOfAllMembersOver(int)
```

혹시나 위의 두 가지 에러가 발생하더라도 로그에서 친절하게 설명해주기 때문에 간단한 수정을 통해 정상동작 확인 가능합니다.

### 테스트

간단한 테스트 코드를 통해 확인해보겠습니다.

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.entity.Member;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import javax.transaction.Transactional;

import static org.junit.jupiter.api.Assertions.assertEquals;

@SpringBootTest
class MemberRepositoryTest {
    @Autowired
    MemberRepository memberRepository;
    
    @Test
    @DisplayName("벌크 업데이트 테스트: 나이가 n살 이상인 멤버의 나이를 1씩 증가시킨다")
    @Transactional
    public void bulkUpdate() {
        // given
        memberRepository.save(new Member("a", 10));
        memberRepository.save(new Member("b", 15));
        memberRepository.save(new Member("c", 20));
        memberRepository.save(new Member("d", 30));
        memberRepository.save(new Member("e", 40));

        // when
        int count = memberRepository.increaseAgeOfAllMembersOver(20);

        // then
        assertEquals(3, count); // (1)
    }
}
```

(1) 20살 이상되는 회원은 3명이기 때문에 업데이트 된 row의 수가 3이어야 합니다.

여기서 주의할 점은 DML 쿼리이기 때문에 반드시 `@Transactional` 애너테이션을 사용해야 한다는 것 입니다.

`@Transactional` 애너테이션 없이 테스트를 수행하게되면 아래와 같은 에러가 발생합니다.

```text
org.springframework.dao.InvalidDataAccessApiUsageException: Executing an update/delete query; nested exception is javax.persistence.TransactionRequiredException: Executing an update/delete query
```

여기서도 `Exception` 이름이 `TransactionRequiredException` 이기 때문에 '아 @Transactional을 빼먹었구나!' 하고 바로 수정할 수 있습니다.

그렇다면 정상 수행시 로그를 확인해보겠습니다.

```text
2021-07-01 20:17:49.958 DEBUG 76312 --- [           main] org.hibernate.SQL                        : 
    insert 
    into
        member
        (age, team_id, username, member_id) 
    values
        (?, ?, ?, ?)
2021-07-01 20:17:49.962 DEBUG 76312 --- [           main] org.hibernate.SQL                        : 
    insert 
    into
        member
        (age, team_id, username, member_id) 
    values
        (?, ?, ?, ?)
2021-07-01 20:17:49.962 DEBUG 76312 --- [           main] org.hibernate.SQL                        : 
    insert 
    into
        member
        (age, team_id, username, member_id) 
    values
        (?, ?, ?, ?)
2021-07-01 20:17:49.962 DEBUG 76312 --- [           main] org.hibernate.SQL                        : 
    insert 
    into
        member
        (age, team_id, username, member_id) 
    values
        (?, ?, ?, ?)
2021-07-01 20:17:49.963 DEBUG 76312 --- [           main] org.hibernate.SQL                        : 
    insert 
    into
        member
        (age, team_id, username, member_id) 
    values
        (?, ?, ?, ?)
2021-07-01 20:17:49.966 DEBUG 76312 --- [           main] org.hibernate.SQL                        : 
    update
        member 
    set
        age=age+1 
    where
        age>=?
```

먼저 5명의 Member를 insert 한 뒤, update 쿼리는 한 번만 발생한 것을 확인할 수 있습니다.

### 주의사항

벌크 업데이트는 영속성 컨텍스트를 통한 `Entity` 관리가 불가능합니다. `save`나 `find` 등을 통해 `repository`에서 가져오게 되면 영속성 컨텍스트로 인식하지만, 직접적으로 update 쿼리를 실행하게되면 영속성 컨텍스트로 인지할 기회가 없기 때문입니다.

이전 테스트를 간단히 수정해 확인해보겠습니다.

```java
    @Test
    @DisplayName("벌크 업데이트 테스트: 나이가 n살 이상인 멤버의 나이를 1씩 증가시킨다")
    @Transactional
    public void bulkUpdate() {
        // given
        memberRepository.save(new Member("a", 10));
        memberRepository.save(new Member("b", 15));
        memberRepository.save(new Member("c", 20));
        memberRepository.save(new Member("d", 30));
        memberRepository.save(new Member("e", 40));
        // when
        int count = memberRepository.increaseAgeOfAllMembersOver(20);
        Member member = memberRepository.findByUsername("e");

        // then
        assertEquals(3, count);
        assertEquals(41, member.getAge());
    }
```

DB에서 다시 조회해 온 `member` 객체는 20살 이상이기 때문에 한 살이 추가된 41살이 되어야 합니다.

하지만 테스트 결과는...

[##_Image|kage@U272p/btq8zy3Msl1/WNZXB66P3n2B2Qzu6WUSuK/img.png|alignCenter|data-origin-width="3166" data-origin-height="572" data-ke-mobilestyle="widthOrigin"|||_##]

초록불이 보이지 않습니다.

그 이유는 위에서 설명한 바와 같고, 그렇다면 아무리 조심해서 사용한다고해도 언제든지 실수할 가능성이 있으니 이를 미연에 방지하는 것이 더 중요하겠죠?

```java
    @Autowired
    EntityManager entityManager; // (1)

    @Test
    @DisplayName("벌크 업데이트 테스트: 나이가 n살 이상인 멤버의 나이를 1씩 증가시킨다")
    @Transactional
    public void bulkUpdate() {
        // given
        memberRepository.save(new Member("a", 10));
        memberRepository.save(new Member("b", 15));
        memberRepository.save(new Member("c", 20));
        memberRepository.save(new Member("d", 30));
        memberRepository.save(new Member("e", 40));
        // when
        int count = memberRepository.increaseAgeOfAllMembersOver(20);

        entityManager.flush(); // (2) 
        entityManager.clear(); // (3)

        Member member = memberRepository.findByUsername("e");

        // then
        assertEquals(3, count);
        assertEquals(41, member.getAge()); 
    }
```

(1) 영속성 컨텍스트를 관리할 수 있는 EntityManager를 주입받습니다.
(2) 남아있는 변동사항이 DB에 반영됩니다.
(3) 영속성 컨텍스트를 비워줍니다.

이렇게 수정한 뒤 다시 실행하면 성공적인 결과를 확인할 수 있습니다.

하지만 당연히 갓프링께서 더 간단한 방법을 지원해주겠죠?

다시 `Repository`로 돌아가 `@Modifying` 애너테이션에 속성을 추가해줍니다.

```java
@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Modifying(clearAutomatically = true)
    @Query("update Member m set m.age = m.age + 1 where m.age >= :age")
    int increaseAgeOfAllMembersOver(@Param("age") int age);
}
```

이렇게 추가해주시면 `EntityManager`를 이용해 수동으로 했던 작업을 자동으로 해줍니다.

`flushAutomatically` 옵션은 `flush()`만, `clearAutomatically` 옵션은 `flush()` 이후 `clear()`까지 호출해줍니다.

[##_Image|kage@Qa1oa/btq8BjxF6qi/YYj76mGLzLLZAzeAVxgdi0/img.png|alignCenter|data-origin-width="1006" data-origin-height="234" width="430" height="100" data-ke-mobilestyle="widthOrigin"|||_##]

테스트는 무사히 통과했지만 그렇다면 안전빵으로 무조건 `clearAutomatically` 옵션을 사용하는 게 나을까요?

답은 "상황별로 다르다" 입니다.

실무에서는 MyBatis와 JPA를 같이 사용하는 경우도 많고, 벌크 업데이트 후 조회가 필요한 상황이 있을 수도 있습니다만 설계에 따라 벌크 업데이트 로직이 별개로 동작한다면 굳이 옵션을 추가할 필요가 없기 때문입니다.

따라서 상황에 맞게 잘 판단해서 사용하는 것이 실수를 방지하고 메모리를 비웠다가 다시 조회하면서 트랜잭션이 추가로 발생하는 상황 또한 방지해줄 수 있습니다.