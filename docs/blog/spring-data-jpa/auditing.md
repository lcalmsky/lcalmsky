![](https://img.shields.io/badge/spring--boot-2.5.1-red) ![](https://img.shields.io/badge/gradle-7.0.2-brightgreen)

> 모든 소스 코드는 [여기](https://github.com/lcalmsky/spring-data-jpa)에서 확인 가능합니다.

`Entity`를 생성하거나 변경할 때, 각각의 시간과 담당자를 추적할 필요가 있습니다.

특히 실무에서 이력을 남기기위해 많이 쓰이는 기능인데요, 이렇게 하기 위해선 `Entity`마다 필요에따라 `생성 시간`, `수정 시간`, `생성한 사람`, `수정한 사람` 이 네 가지 속성을 가져야 합니다.

`JPA`를 사용할 경우 `Entity`가 결국 객체이기 때문에 공통적인 속성을 모두 가지는 클래스를 설계하면 간단히 해결할 수 있는데요, 그 방법을 한 번 살펴보겠습니다.

### BaseEntity 정의

생성 시간, 수정 시간을 가지는 `Entity`를 작성합니다.

```java
package io.lcalmsky.springdatajpa.domain.entity;

import javax.persistence.Column;
import javax.persistence.MappedSuperclass;
import javax.persistence.PrePersist;
import javax.persistence.PreUpdate;
import java.time.LocalDateTime;

@MappedSuperclass // (1)
public class BaseEntity {
    @Column(updatable = false) // (2)
    private LocalDateTime createTime;
    private LocalDateTime updateTime;

    @PrePersist // (3)
    public void before() {
        LocalDateTime now = LocalDateTime.now();
        this.createTime = now;
        this.updateTime = now;
    }

    @PreUpdate // (4)
    public void always() {
        this.updateTime = LocalDateTime.now();
    }
}
```

> (1) `@MappedSuperClass`는 상속된 엔티티에 매핑정보가 적용되도록 지정합니다. 이 클래스에 자체에 대해 매핑되는 테이블이 존재하지 않기 때문에 하위 클래스의 테이블에만 영향을 미칩니다. 하위 클래스에서는 `@AttributeOverride`나 `@AssociationOverride`로 재정의 할 수 있습니다. <br>
> (2) `updatable = false`로 지정하면 실수로 값을 바꿔도 업데이트 되지 않습니다.<br>
> (3) `@PrePersist`는 persist(insert) 하기 전에 호출합니다.<br>
> (4) `@PreUpdate`는 update하기 전에 호출합니다.<br>

이제 `Member Entity`가 `Base Entity`를 상속하게 합니다.

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
@NamedEntityGraph(name = "member.findAll", attributeNodes = @NamedAttributeNode("team"))
public class Member extends BaseEntity { // (1) 
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
    // 생략
}
```

> (1) `BaseEntity`를 상속합니다.

이렇게 바꿔놓고 아무 테스트나 실행하기 전에 설정파일에 `spring.jpa.hibernate.ddl-auto`가 `create`로 설정되어있는지 확인합니다. (테스트 환경이기 때문에 Entity의 변경 사항이 바로 테이블 스키마에 적용되는지 확인)

이제 해당 데이터들이 잘 들어가고있는지 확인하기 위한 테스트를 작성합니다.

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.entity.Member;
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
    public void baseEntityTest() throws InterruptedException {
        // given
        Member member = new Member("홍길동", 20);
        memberRepository.save(member); // (1)
        Thread.sleep(500); // (2)
        member.changeUsername("이순신");
        entityManager.flush(); // (3)
        entityManager.clear();
        // when
        Member dbMember = memberRepository.findById(member.getId()).orElseThrow(IllegalArgumentException::new);
        // then
        System.out.println(dbMember.getCreateTime());
        System.out.println(dbMember.getUpdateTime());
    }
}
```

> (1) `@PrePersist` 애너테이션이 추가되어있는 메서드가 실행됩니다.
> (2) 생성 시간과 수정 시간을 구분하기 위해 잠시 텀을 둡니다.
> (3) `@PreUpdate` 에너테이션이 추가되어있는 메서드가 실행됩니다.

테스트를 실행한 결과는 아래와 같습니다.

```text
2021-07-09 03:13:45.387 DEBUG 25042 --- [           main] org.hibernate.SQL                        : 
    
    create table member (
       member_id bigint not null,
        create_time timestamp,
        update_time timestamp,
        age integer not null,
        username varchar(255),
        team_id bigint,
        primary key (member_id)
    ) 
2021-07-09 03:25:19.822 DEBUG 25125 --- [           main] org.hibernate.SQL                        : 
    insert 
    into
        member
        (create_time, update_time, age, team_id, username, member_id) 
    values
        (?, ?, ?, ?, ?, ?)
2021-07-09 03:25:19.822 DEBUG 25125 --- [           main] org.hibernate.SQL                        : 
    insert 
    into
        member
        (create_time, update_time, age, team_id, username, member_id) 
    values
        (?, ?, ?, ?, ?, ?)
2021-07-09 03:25:19.868 DEBUG 25125 --- [           main] org.hibernate.SQL                        : 
    select
        member0_.member_id as member_i1_0_0_,
        member0_.create_time as create_t2_0_0_,
        member0_.update_time as update_t3_0_0_,
        member0_.age as age4_0_0_,
        member0_.team_id as team_id6_0_0_,
        member0_.username as username5_0_0_ 
    from
        member member0_ 
    where
        member0_.member_id=?
// 생략
2021-07-09T03:25:19.256415
2021-07-09T03:25:19.808679
```

이렇게 테이블 스키마가 바뀐 채로 생성되고, 하나의 `Member`를 `insert`하고 `update`한 뒤 다시 `select` 해오는 쿼리와 생성, 수정 시간을 제대로 출력하는 것을 확인할 수 있습니다.

위에서 `@MappedSuperClass`를 생략하게 되면 바뀐 스키마가 적용되지 않으니 꼭 잘 챙겨줘야 합니다.

### Auditing 사용

`BaseEntity`를 정의하고 필요한 `Entity`마다 `BaseEntity`를 상속받게 한 것 만으로도 상당히 편해졌지만 `스프링 데이터 JPA`에서는 훨씬 더 간단한 기능을 제공합니다.

먼저 기본 설정을 불러올 수 있는 애너테이션을 추가해줘야 합니다.

```java
package io.lcalmsky.springdatajpa;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

@SpringBootApplication
@EnableJpaAuditing // (1)
public class SpringDataJpaApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringDataJpaApplication.class, args);
    }

}
```

> (1) Auditing 사용을 편리하게 해주는 애너테이션으로 사용했을 때 기본 설정들을 불러옵니다.

다음은 `Auditing`을 사용할 `Entity`를 작성합니다.

```java
package io.lcalmsky.springdatajpa.domain.entity;

import lombok.Getter;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.Column;
import javax.persistence.EntityListeners;
import javax.persistence.MappedSuperclass;
import java.time.LocalDateTime;

@EntityListeners(AuditingEntityListener.class) // (1)
@MappedSuperclass
@Getter
public class AuditingEntity {
    @CreatedDate // (2)
    @Column(updatable = false)
    private LocalDateTime createdDate;
    @LastModifiedDate // (3)
    private LocalDateTime lastModifiedDate;
}
```

> (1) 이벤트가 발생할 때 호출해줄 Listener 클래스를 지정합니다. <br>
> (2) 생성 날짜를 나타내는 필드로 선언합니다. <br>
> (3) 수정 날짜를 나타내는 필드로 선언합니다. <br>

`Member Entity`가 `AuditingEntity`를 상속하게 수정한 뒤 테스트 클래스도 수정해줍니다.

```java
public class Member extends AuditingEntity {
    // 생략
}
```

```java
public void baseEntityTest()throws InterruptedException{
    // given
    Member member=new Member("홍길동",20);
    memberRepository.save(member);
    Thread.sleep(500);
    member.changeUsername("이순신");
    entityManager.flush();
    entityManager.clear();
    // when
    Member dbMember=memberRepository.findById(member.getId()).orElseThrow(IllegalArgumentException::new);
    // then
    System.out.println(dbMember.getCreatedDate());
    System.out.println(dbMember.getLastModifiedDate());
}
```

필드 이름을 변경하였기 때문에 출력하는 부분을 수정하였습니다.

다시 테스트를 수행해보면,

```text
2021-07-09 03:40:59.989 DEBUG 25266 --- [           main] org.hibernate.SQL                        : 
    
    create table member (
       member_id bigint not null,
        created_date timestamp,
        last_modified_date timestamp,
        age integer not null,
        username varchar(255),
        team_id bigint,
        primary key (member_id)
    )
// 생략
2021-07-09T03:41:02.245141
2021-07-09T03:41:02.852973
```

테이블 스키마가 변경되었고 시간을 출력하는 기능도 정상동작 하였음을 확인할 수 있습니다.

이제 마지막으로 생성, 수정한 사람의 이력을 남기는 기능을 추가해보겠습니다.

```java
package io.lcalmsky.springdatajpa.domain.entity;

import lombok.Getter;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.Column;
import javax.persistence.EntityListeners;
import javax.persistence.MappedSuperclass;
import java.time.LocalDateTime;

@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
@Getter
public class AuditingEntity {
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;
    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
    @CreatedBy // (1)
    @Column(updatable = false)
    private String createdBy;
    @LastModifiedBy // (2)
    private String lastModifiedBy;
}
```

> (1) 누구에 의해 생성했는지 지정하는 필드로 선언합니다.
> (2) 누구에 의해 변경되었는지 지정하는 필드로 선언합니다.

이렇게 수정은 했을 때 과연 `createdBy`, `lastModifiedBy`에는 어떤 값이 들어갈까요?

이 부분은 `AuditingAware<T>`를 등록하여 직접 정의할 수 있습니다.

```java
@Bean
public AuditorAware<String> auditorProvider() {
    return () -> Optional.of(UUID.randomUUID().toString());
}
```

`@Configuration` 클래스나 `@Configuration`이 포함된 클래스에 `AuditorAware`를 `Bean`으로 등록하시면 빈의 구현 내용에 따라 `Auditor`를 추출해 해당 필드에 매핑시켜 값을 업데이트하게 됩니다.

예시라 간단히 UUID를 생성했지만, 실무에서 많이 쓰이는 방식은 `스프링 부트 security`와 결합하여 `Authentication` 정보를 가져와 거기 존재하는 사용자의 아이디나 인증된 정보를 포함시키는 방식으로 사용합니다.

그리고 `Bean`으로 등록할 수 있다는 건 `Component`로 등록해도 상관 없다는 뜻이겠죠?

따라서 하나의 클래스로 따로 작성하셔도 상관 없습니다.

> 현재 예시로 만든 프로젝트에는 security 패키지 의존성이 추가되어있지 않아 다음에 기회가 되면 업데이트 하려고 합니다.

아무튼, 이렇게 `AuditorAware` `Bean`을 정의했다면 다시 테스트를 수행해보겠습니다.

마지막에 두 줄을 추가하여 `Auditor`를 제대로 출력하는지 확인해보겠습니다.

```java
@Test
@Transactional
public void baseEntityTest()throws InterruptedException{
    // given
    Member member=new Member("홍길동",20);
    memberRepository.save(member); // (1)
    Thread.sleep(500); // (2)
    member.changeUsername("이순신");
    entityManager.flush(); // (3)
    entityManager.clear();
    // when
    Member dbMember=memberRepository.findById(member.getId()).orElseThrow(IllegalArgumentException::new);
    // then
    System.out.println(dbMember.getCreatedDate());
    System.out.println(dbMember.getLastModifiedDate());
    System.out.println(dbMember.getCreatedBy());
    System.out.println(dbMember.getLastModifiedBy());
}
```

```text
2021-07-09 03:56:21.745 DEBUG 25387 --- [           main] org.hibernate.SQL                        : 
    
    create table member (
       member_id bigint not null,
        created_by varchar(255),
        created_date timestamp,
        last_modified_by varchar(255),
        last_modified_date timestamp,
        age integer not null,
        username varchar(255),
        team_id bigint,
        primary key (member_id)
    )
```

일단 스키마는 잘 변경되었고,

```text
2021-07-09T03:56:24.074478
2021-07-09T03:56:24.630102
1fa25221-4e0b-4a69-bea8-9ff69ffa279d
50fb1710-ad0f-4011-ad45-b06e49d02ceb
```

마지막에 출력된 결과도 UUID로 랜덤 생성하였기 때문에 `createdBy`와 `lastModifiedBy`가 서로 다른 것을 확인할 수 있습니다.

---

일반적으로 실무에서는 생성 시간, 수정 시간에 대해서는 대부분의 테이블에서 공통적으로 사용하지만 생성자, 수정자가 필수로 필요한 테이블은 많지 않습니다.

따라서 시간 관련 필드만 가지는 `Entity`를 만들고 해당 `Entity`를 `AuditingEntity`가 상속하게 만들면 필요한 테이블에서 자유롭게 추가할 수 있습니다.

이는 자바 설계관점으로 생각하여 얼마든지 자유롭게 수정될 수 있으므로 상황에 맞게 사용하시면 됩니다.