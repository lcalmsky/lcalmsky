> 모든 소스 코드는 [여기](https://github.com/lcalmsky/spring-data-jpa)에서 확인 가능합니다.

그동안 API에서 페이징 처리를 위해 page와 size를 받아 공식을 계산해서 SQL문을 수정해보신 경험이 있을 겁니다.

스프링 데이터 JPA는 매우 편리하게 표준화된 페이징 및 정렬 방식을 제공합니다.

### 페이징과 정렬 파라미터

* `org.springframework.data.domain.Sort`: 정렬 인터페이스
* `org.springframework.data.domain.Pageable`: 페이징 인터페이스(내부에 `Sort` 포함)

### 반환 타입

* `org.springframework.data.domain.Page`: total count를 포함하는 페이징 결과 타입, count 쿼리를 추가로 수행
* `org.springframework.data.domain.Slice`: total count를 포함하지 않는 페이징 결과 타입, count 쿼리를 추가로 수행하지 않고 다음 페이지만 확인 가능
* `java.util.List`: count 쿼리 없이 결과만 반환

아래와 같은 방식으로 Repository 인터페이스 내에서 사용 합니다.

```java
Page<Member> findByUsername(String username,Pageable pageable); // + count 쿼리
Slice<Member> findByUsername(String username,Pageable pageable);
List<Member> findByUsername(String username,Pageable pageable);
List<Member> findByUsername(String username,Sort sort); // 정렬만 적용
```

### 예제

이제 예제 소스 코드 및 실행 결과를 확인해 봅시다.

먼저 Page를 반환하는 테스트 소스 코드 입니다.

<details><summary>Member.java</summary>

```java
package io.lcalmsky.springdatajpa.domain.entity;

import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.ToString;

import javax.persistence.*;

@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;
    private String username;
    private int age;
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    @ToString.Exclude
    private Team team;

    public Member(String username, int age, Team team) {
        this.username = username;
        this.age = age;
        if (team != null) {
            changeTeam(team);
        }
    }

    public Member(String username, int age) {
        this(username, age, null);
    }

    public void changeTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
    }
}
```

</details>

<details><summary>Team.java</summary>

```java
package io.lcalmsky.springdatajpa.domain.entity;

import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.ToString;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString
public class Team {
    @Id
    @GeneratedValue
    @Column(name = "team_id")
    private Long id;
    private String name;
    @OneToMany(mappedBy = "team")
    @ToString.Exclude
    private List<Member> members = new ArrayList<>();

    public Team(String name) {
        this.name = name;
    }

    public void setMembers(List<Member> members) {
        this.members = members;
    }
}
```

</details>

<details><summary>MemberRepository.java</summary>

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.dto.MemberDto;
import io.lcalmsky.springdatajpa.domain.entity.Member;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.Collection;
import java.util.List;

@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);

    @Query("select m from Member m where m.username = :username and m.age = :age")
    List<Member> findUser(@Param("username") String username, @Param("age") int age);

    @Query("select m.username from Member m")
    List<String> findUsernameList();

    @Query("select new io.lcalmsky.springdatajpa.domain.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
    List<MemberDto> findMemberDto();

    @Query("select m from Member m where m.username in :names")
    List<Member> findByNames(@Param("names") Collection<String> names);

    Page<Member> findByAge(int age, Pageable pageable);
}
```

</details>

잘 동작하는지 확인하기위해 먼저 조회한 데이터를 출력하는 테스트를 작성했습니다.

MemberRepositoryTest.java

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.entity.Member;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Sort;

import java.util.List;

@SpringBootTest
class MemberRepositoryTest {
    @Autowired
    MemberRepository memberRepository;

    @Test
    @DisplayName("페이징 테스트")
    void paging() {
        // given
        memberRepository.save(new Member("a", 10));
        memberRepository.save(new Member("b", 10));
        memberRepository.save(new Member("c", 10));
        memberRepository.save(new Member("d", 10));
        memberRepository.save(new Member("e", 10));

        // when
        PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));
        Page<Member> memberPages = memberRepository.findByAge(10, pageRequest);

        // then
        List<Member> content = memberPages.getContent();
        long totalElements = memberPages.getTotalElements();
        System.out.println("content = " + content);
        System.out.println("totalElements = " + totalElements);
    }
}
```

테스트를 수행하면서 수행된 쿼리 로그를 먼저 살펴보면 

```sql
select
    member0_.member_id as member_i1_0_,
    member0_.age as age2_0_,
    member0_.team_id as team_id4_0_,
    member0_.username as username3_0_ 
from
    member member0_ 
where
    member0_.age=? 
order by
    member0_.username desc limit ?
```

이렇게 Member를 가져오는 쿼리 1회,

```sql
select
    count(member0_.member_id) as col_0_0_ 
from
    member member0_ 
where
    member0_.age=?
```

count를 가져오는 쿼리 1회로 구성되어 있습니다.

첫 번 째 쿼리의 경우 PageRequest에 시작 page를 0으로 주었기 때문에 offset 값은 존재하지 않고, size를 3으로 지정하였기 때문에 limit 3을 지정하여 쿼리합니다.

정렬 옵션도 제대로 적용된 것을 확인할 수 있습니다.

두 번 째 쿼리의 경우 `totalPage`를 얻기위해 count 함수를 사용합니다.

최적화를 위해 필요 없는 정렬 관련된 구문은 자동으로 제외시키는 등 나름 최적화 된 방식을 제공합니다.

실제 수행된 결과는 아래와 같습니다.

```text
content = [Member(id=5, username=e, age=10), Member(id=4, username=d, age=10), Member(id=3, username=c, age=10)]
totalElements = 5
```

이제 실제 검증하기위한 소스 코드로 수정해보겠습니다.

MemberRepositoryTest.java

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.entity.Member;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Sort;

import java.util.List;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;

@SpringBootTest
class MemberRepositoryTest {
    @Autowired
    MemberRepository memberRepository;

    @Test
    @DisplayName("페이징 테스트")
    void paging() {
        // given
        memberRepository.save(new Member("a", 10));
        memberRepository.save(new Member("b", 10));
        memberRepository.save(new Member("c", 10));
        memberRepository.save(new Member("d", 10));
        memberRepository.save(new Member("e", 10));

        // when
        PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));
        Page<Member> memberPages = memberRepository.findByAge(10, pageRequest);

        // then
        List<Member> content = memberPages.getContent();
        long totalElements = memberPages.getTotalElements();

        assertEquals(3, content.size()); // (1)
        assertEquals(5, totalElements); // (2)
        assertEquals(0, memberPages.getNumber()); // (3)
        assertEquals(2, memberPages.getTotalPages()); // (4)
        assertTrue(memberPages.isFirst()); // (5)
        assertTrue(memberPages.hasNext()); // (6)
    }
}
```

사전에 5개의 Member 객체를 저장한 뒤 첫 페이지(0), 페이지 크기 3으로 조회하였습니다.

(1) 반환한 내용 갯수가 3개인지 확인합니다.
(2) 전체 내용 갯수가 5개인지 확인합니다.
(3) 현재 페이지를 반환합니다. 
(4) 전체 페이지가 2인지 확인합니다. 5개의 내용을 3개의 사이즈로 나누었기 때문에 2페이지가 됩니다.
(5) 첫 번 째 페이지인지 확인합니다. 0번으로 조회하였으니 true 입니다.
(6) 다음 페이지가 있는지 확인합니다. 0~1까지 페이지가 존재하고 현재는 0번 페이지 이므로 true 입니다.


다음은 Slice를 반환하는 소스 코드 입니다.

Slice는 Page의 부모 인터페이스 입니다.

따라서

```java
Page<Member> page = memberRepository.findByAge(10, pageRequest);
Slice<Member> slice = page;
```

이렇게도 사용할 수 있습니다.

하지만 이렇게 된다면 전체 카운트를 하지 않는 Slice를 사용할 이유가 없어지므로 애초에 Slice를 반환하는 메서드를 호출해야 합니다.

보통 Page, Slice 어떤 것을 반환하든 동일한 쿼리 기준으로 메서드 시그니처 또한 동일하기 때문에(맨 위의 예시 참조) 실수하기 쉬우므로 반드시 제대로 확인해야 합니다.

저는 두 메서드 구분을 위해 이름을 다르게하여 Slice를 반환하는 메서드를 추가하였습니다.

<details><summary>MemberRepository.java</summary>

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.dto.MemberDto;
import io.lcalmsky.springdatajpa.domain.entity.Member;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Slice;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.Collection;
import java.util.List;

@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);

    @Query("select m from Member m where m.username = :username and m.age = :age")
    List<Member> findUser(@Param("username") String username, @Param("age") int age);

    @Query("select m.username from Member m")
    List<String> findUsernameList();

    @Query("select new io.lcalmsky.springdatajpa.domain.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
    List<MemberDto> findMemberDto();

    @Query("select m from Member m where m.username in :names")
    List<Member> findByNames(@Param("names") Collection<String> names);

    Page<Member> findByAge(int age, Pageable pageable);

    Slice<Member> findSlicesByAge(int age, Pageable pageable);
}
```

</details>

테스트 소스 코드도 추가해 줍니다.

```java
@Test
@DisplayName("페이징 테스트(slice)")
void slice() {
    // given
    memberRepository.save(new Member("a", 10));
    memberRepository.save(new Member("b", 10));
    memberRepository.save(new Member("c", 10));
    memberRepository.save(new Member("d", 10));
    memberRepository.save(new Member("e", 10));

    // when
    PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));
    Slice<Member> memberPages = memberRepository.findSlicesByAge(10, pageRequest); // Slice를 반환하는 메서드 호출

    // then
    List<Member> content = memberPages.getContent();

    assertEquals(3, content.size());
    assertEquals(0, memberPages.getNumber());
    assertTrue(memberPages.isFirst());
    assertTrue(memberPages.hasNext());
}
```

위 테스트를 수행한 뒤 로그를 살펴보면 count 쿼리가 수행되지 않는 것을 확인할 수 있습니다.

Slice의 경우 모바일 디바이스에서 많이 쓰는 더보기 방식에 적합합니다.

다음은 List를 반환하는 예제 입니다.

페이징에 필요한 다른 특징들 외에 단순히 offset, limit만 사용하는 경우 List를 반환하는 것이 적합합니다.

(글이 너무 길어지니 수정한 부분만 보여드리겠습니다.)

MemberRepository.java

```java
List<Member> findMembersByAge(int age, Pageable pageable);
```

MemberTest.java

```java
@Test
@DisplayName("페이징 테스트(list)")
void list() {
    // given
    memberRepository.save(new Member("a", 10));
    memberRepository.save(new Member("b", 10));
    memberRepository.save(new Member("c", 10));
    memberRepository.save(new Member("d", 10));
    memberRepository.save(new Member("e", 10));

    // when
    PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));
    List<Member> memberPages = memberRepository.findMembersByAge(10, pageRequest);

    // then
    assertEquals(3, memberPages.size());
}
```

List를 반환했기 때문에 페이징 관련된 항목을 추가로 검증할 순 없고, 요청한 대로 3개를 가져왔는지 확인하여 정상적으로 수행됨을 알 수 있습니다.

### 주의할 점

페이징에서 사용되는 total count를 획득하는 부분이 실제 DB에 부하를 많이 줍니다. 특히 실무에서 사용될 경우 데이터 갯수가 어마어마하게 많아질 수 있고 Join 등이 포함되어있는 경우 의미 없는 카운트가 될 수 있습니다.

다행히도 스프링 데이터 JPA는 `countQuery`를 분리할 수 있는 방법을 제공합니다.

MemberRepository.java

```java
@Query(value = "select m from Member m left join Team t",
        countQuery = "select count(m) from Member m")
Page<Member> findMembers(Pageable pageable);
```

### 페이징한 Entity를 DTO로 매핑하기

API를 설계할 때 Entity를 그대로 반환하는 경우는 아주아주 드물게 존재하긴 합니다만 되도록이면 지양하고 있습니다.

페이징한 결과가 `Page<Entity>` 일 때 원하는 객체로 매핑하는 방법을 소개합니다.

`Page` 인터페이스는 `map`이라는 메서드를 제공합니다. `Stream`이나 `Optional`에서의 `map`과 동일한 `Function<T, R>`을 구현하시면 됩니다.

Member 페이지를 MemberDto 페이지로 매핑하는 예제 소스 코드 입니다. 

MemberDto.java
```java
package io.lcalmsky.springdatajpa.domain.dto;

import lombok.AccessLevel;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor
public class MemberDto {
    private Long id;
    private String username;
    private String teamName;
}
```

MemberRepositoryTest.java
```java
@Test
@DisplayName("페이징 후 매핑 테스트")
void pagingWithMapping() {
    // given
    memberRepository.save(new Member("a", 10));
    memberRepository.save(new Member("b", 10));
    memberRepository.save(new Member("c", 10));
    memberRepository.save(new Member("d", 10));
    memberRepository.save(new Member("e", 10));

    // when
    PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));
    Page<Member> memberPages = memberRepository.findByAge(10, pageRequest);
    Page<MemberDto> memberDtoPages = memberPages.map(m -> new MemberDto(m.getId(), m.getUsername(), m.getTeam().getName()));
    memberDtoPages.stream().forEach(System.out::println);
}
```

```text
MemberDto(id=5, username=e, teamName=null)
MemberDto(id=4, username=d, teamName=null)
MemberDto(id=3, username=c, teamName=null)
```

Page 인터페이스를 유지하면서 내부 content만 Entity(Member)에서 DTO(MemberDto)로 변환된 걸 확인할 수 있습니다.