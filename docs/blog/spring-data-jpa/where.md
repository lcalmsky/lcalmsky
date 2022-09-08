DB에서 물리적으로 데이터를 지우는 것이 아니라 논리적으로 삭제하는 방법이 있습니다.

많이 사용하는 방법 중 하나가 바로 삭제 여부를 판단하는 컬럼을 사용하는 것인데요, 삭제 된 날짜가 존재하면 정확한 삭제 시기를 알 수 있으므로 `deleted_at`과 같은 컬럼을 사용할 수 있습니다.

하지만 이런 컬럼이 존재할 경우 정상 데이터를 조회하기 위한 모든 쿼리에 `where deleted_at is null`과 같은 조건절이 필요합니다.

이럴 때 `@Where` 애너테이션을 활용하면 간단히 해결할 수 있습니다.

먼저 `BaseEntity`를 생성해줍니다.

```java
package io.lcalmsky.wheredemo;

import java.time.LocalDateTime;
import javax.persistence.Column;
import javax.persistence.EntityListeners;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.MappedSuperclass;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

@MappedSuperclass
@EntityListeners(value = {AuditingEntityListener.class})
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public abstract class BaseEntity {

  @Getter
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  @Column(name = "id")
  private Long id;

  @Getter
  @CreatedDate
  @Column(name = "created_at",
      columnDefinition = "datetime null default null")
  private LocalDateTime createdAt;

  @Getter
  @LastModifiedDate
  @Column(name = "updated_at",
      columnDefinition = "datetime null default null")
  private LocalDateTime updatedAt;

  @Getter(value = AccessLevel.PROTECTED)
  @Column(name = "deleted_at",
      columnDefinition = "datetime null default null")
  private LocalDateTime deletedAt;

  protected BaseEntity(Long id) {
    this.id = id;
  }

  public void deleteSoftly(LocalDateTime deletedAt) {
    this.deletedAt = deletedAt;
  }

  public boolean isSoftDeleted() {
    return null != deletedAt;
  }

  public void undoDeletion() {
    this.deletedAt = null;
  }
}
```

앞으로 모든 `Entity`가 `id`와 `deleted_at`, `auditing` 관련 컬럼들을 사용하기 위해 `@MappedSupperclass`를 사용해 `BaseEntity`를 정의하였습니다.

다음으로 `BaseEntity`를 상속받는 `Member` `Entity`를 구현합니다.

```java
package io.lcalmsky.wheredemo;

import javax.persistence.Entity;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.ToString;

@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@ToString
public class Member extends BaseEntity {

  private String name;
}

```

어차피 테스트 할 내용은 `deleted_at` 컬럼과 관련이 있기 때문에 name 컬럼 하나만 가지는 `Entity`를 작성하였습니다.

다음으로 테스트하기 위해 `MemberRepository`를 구현합니다.

```java
package io.lcalmsky.wheredemo;

import java.util.Optional;
import org.springframework.data.jpa.repository.JpaRepository;

public interface MemberRepository extends JpaRepository<Member, Long> {

  Optional<Member> findByName(String name);

  Optional<Member> findByNameAndDeletedAtIsNull(String name);

}
```

단순하게 이름만으로 찾는 쿼리 메서드와 이름+삭제여부로 찾는 쿼리 메서드 두 개를 구현하였습니다.

아직 `@Where` 애너테이션을 사용하기 전이지만 결과를 확인해보겠습니다.

```java
package io.lcalmsky.wheredemo;

import static org.junit.jupiter.api.Assertions.assertTrue;

import java.time.LocalDateTime;
import java.util.Optional;
import java.util.stream.Stream;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;

@DataJpaTest
class MemberRepositoryTest {

  private static final String[] REGISTERED_MEMBER = {"kane", "son", "kulusevski", "recharlison"};

  private static final String[] UNREGISTERED_MEMBER = {"bergwijn", "ndombélé"};

  @Autowired
  MemberRepository memberRepository;

  @BeforeEach
  void setup() {
    Stream.of(REGISTERED_MEMBER)
        .map(Member::of)
        .forEach(memberRepository::save);
    Stream.of(UNREGISTERED_MEMBER)
        .map(Member::of)
        .peek(m -> m.deleteSoftly(LocalDateTime.now()))
        .forEach(memberRepository::save);
  }

  @AfterEach
  void teardown() {
    memberRepository.deleteAll();
  }

  @Test
  @DisplayName("@Where 애너테이션 사용하지 않고 이름으로 로스터의 member를 찾음")
  void test() {
    Optional<Member> kane = memberRepository.findByName("kane");
    Optional<Member> son = memberRepository.findByNameAndDeletedAtIsNull("son");
    Optional<Member> bergwijn = memberRepository.findByName("bergwijn");
    Optional<Member> ndombélé = memberRepository.findByNameAndDeletedAtIsNull("ndombélé");
    assertTrue(kane.isPresent());
    assertTrue(son.isPresent());
    assertTrue(bergwijn.isPresent());
    assertTrue(ndombélé.isEmpty());
  }
}
```

토트넘의 등록된 선수들과 방출된 선수들을 모두 저장한 뒤 테스트를 해보았습니다.

테스트는 모두 성공하였고, 은돔벨레의 경우 `deletedAt`이 `null`이 아니므로 `empty`라는 원하는 결과가 나왔지만 베르흐바인의 경우 여전히 존재(`present`)하고 있음을 확인할 수 있습니다.

이제 `Entity`에 `@Where` 애너테이션을 추가하면서 전체적으로 코드를 수정해보겠습니다.

```java
package io.lcalmsky.wheredemo;

import javax.persistence.Entity;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.ToString;
import org.hibernate.annotations.Where;

@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@ToString
@Where(clause = "deleted_at is null")
public class Member extends BaseEntity {

  private String name;

  private Member(String name) {
    this.name = name;
  }

  public static Member of(String name) {
    return new Member(name);
  }
}
```

클래스 바로 위에 `@Where` 애너테이션을 이용해 조건절에 들어갈 내용을 추가하였습니다.

다음으로 `MemberRepository`에서 기존에 삭제 여부 `where` 조건이 포함된 메서드를 삭제해보겠습니다.

```java
package io.lcalmsky.wheredemo;

import java.util.Optional;
import org.springframework.data.jpa.repository.JpaRepository;

public interface MemberRepository extends JpaRepository<Member, Long> {

  Optional<Member> findByName(String name);
  
}
```

마지막으로 테스트를 수정해보겠습니다.

```java
// 생략
@DataJpaTest
class MemberRepositoryTest {
  // 생략
  @Test
  @DisplayName("@Where 애너테이션 사용하지 않고 이름으로 member를 찾음")
  void test() {
    Optional<Member> kane = memberRepository.findByName("kane");
    Optional<Member> son = memberRepository.findByName("son");
    Optional<Member> bergwijn = memberRepository.findByName("bergwijn");
    Optional<Member> ndombélé = memberRepository.findByName("ndombélé");
    assertTrue(kane.isPresent());
    assertTrue(son.isPresent());
    assertTrue(bergwijn.isEmpty());
    assertTrue(ndombélé.isEmpty());
  }
}
```

베르흐바인과 은돔벨레는 조회되지 않는 것을 검증하도록 수정하였고 테스트는 잘 통과되었습니다.

로그를 살펴보면

```text
Hibernate: 
    select
        member0_.id as id1_0_,
        member0_.created_at as created_2_0_,
        member0_.deleted_at as deleted_3_0_,
        member0_.updated_at as updated_4_0_,
        member0_.name as name5_0_ 
    from
        member member0_ 
    where
        (
            member0_.deleted_at is null
        ) 
        and member0_.name=?
2022-09-09 02:22:39.259 TRACE 15583 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [bergwijn]
Hibernate: 
    select
        member0_.id as id1_0_,
        member0_.created_at as created_2_0_,
        member0_.deleted_at as deleted_3_0_,
        member0_.updated_at as updated_4_0_,
        member0_.name as name5_0_ 
    from
        member member0_ 
    where
        (
            member0_.deleted_at is null
        ) 
        and member0_.name=?
2022-09-09 02:22:39.260 TRACE 15583 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [ndombélé]
```

`deleted_at is null`이 `where` 절에 잘 포함되어 있는 것을 확인할 수 있습니다.

그렇다면 마지막으로, `deleted_at is not null`과 같이 해당 컬럼에 대해 다른 조건을 쿼리 메서드(또는 JPQL)로 추가하면 어떻게 될까요?

```java
package io.lcalmsky.wheredemo;

import java.util.Optional;
import org.springframework.data.jpa.repository.JpaRepository;

public interface MemberRepository extends JpaRepository<Member, Long> {
  
  Optional<Member> findByName(String name);

  Optional<Member> findByNameAndDeletedAtIsNotNull(String name);
  
}
```

`MemberRepository`에 상반되는 조건을 가진 쿼리 메서드를 추가했습니다.

```java
// 생략
@DataJpaTest
class MemberRepositoryTest {
  // 생략
  @Test
  @DisplayName("@Where 애너테이션 사용하여 이름으로 로스터에 등록된 member를 찾음")
  void test() {
    Optional<Member> kane = memberRepository.findByName("kane");
    Optional<Member> son = memberRepository.findByName("son");
    Optional<Member> bergwijn = memberRepository.findByNameAndDeletedAtIsNotNull("bergwijn");
    Optional<Member> ndombélé = memberRepository.findByNameAndDeletedAtIsNotNull("ndombélé");
    assertTrue(kane.isPresent());
    assertTrue(son.isPresent());
    assertTrue(bergwijn.isPresent());
    assertTrue(ndombélé.isPresent());
  }
}
```

기대한 결과는 `deleted_at is not null`이 쿼리에 포함되면서 베르흐바인, 은돔벨레 모두 검색이 되는 것이었는데, 테스트는 실패하였습니다.

로그를 확인해보니

```text
Hibernate: 
    select
        member0_.id as id1_0_,
        member0_.created_at as created_2_0_,
        member0_.deleted_at as deleted_3_0_,
        member0_.updated_at as updated_4_0_,
        member0_.name as name5_0_ 
    from
        member member0_ 
    where
        (
            member0_.deleted_at is null
        ) 
        and member0_.name=? 
        and (
            member0_.deleted_at is not null
        )
2022-09-09 02:25:47.207 TRACE 15642 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [bergwijn]
```

`@Where` 절에 설정한 조건이 먼저 포함되어버리면서 쿼리가 의도한대로 동작하지 않았습니다.

그렇다면 `@Where` 애너테이션을 무시하기 위해선 어떻게 해야할까요?

```java
@Query(value = "select * from member where deleted_at is not null", nativeQuery = true)
Optional<Member> findByNameAndDeletedAtIsNotNull(String name);
```

이런식으로 `nativeQuery` 옵션을 사용할 수 있습니다.

하지만 이렇게 할 경우 JPA의 큰 장점 중 하나인 SQL을 변경할 수 있다는 점을 활용하기 어려워집니다.

따라서 `@Where` 애너테이션은 꼭 필요한 경우에만 사용해야 합니다.

참고로 관계 설정이 되어있는 경우에도 동작하니 더 유의해야겠습니다😄