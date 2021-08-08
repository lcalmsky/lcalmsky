![](https://img.shields.io/badge/spring--boot-2.5.1-red) ![](https://img.shields.io/badge/gradle-7.0.2-brightgreen)

> 모든 소스 코드는 [여기](https://github.com/lcalmsky/spring-data-jpa)에서 확인할 수 있습니다.

바로 [이전 포스팅](https://jaime-note.tistory.com/62)에서 `JPA`가 `save()`를 호출할 때 새로운 `Entity`인지 기존 `Entity`인지 판단해 `persist()`
또는 `merge()`를 호출한다는 내용을 다룬 바 있습니다.

그렇다면 JPA는 과연 어떻게 새로운 `Entity`인지 판단하는 것 일까요?

설명을 위해 간단한 소스 코드를 작성해보겠습니다.

```java
package io.lcalmsky.springdatajpa.domain.entity;

import lombok.Getter;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
@Getter
public class Item {
    @Id
    @GeneratedValue
    private Long id;
}
```

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.entity.Item;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ItemRepository extends JpaRepository<Item, Long> {
}
```

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.entity.Item;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class ItemRepositoryTest {
    @Autowired
    ItemRepository itemRepository;

    @Test
    void save() {
        Item item = new Item();
        itemRepository.save(item);
    }
}
```

먼저 간단하게 `ID`만 가지는 `Entity`인 `Item`을 작성하고 해당 `Entity`를 관리하기 위해 `ItemRepository`를 작성한 뒤 테스트 클래스로 `Item` `Entity`를 `ItemRepository`에 저장해보았습니다.

테스트 코드에서 `Item` 객체를 생성한 뒤 `ID`를 직접 세팅해주지 않았지만 `Entity` 내에 `@GeneratedValue` 애너테이션에 의해 `ID` 값이 자동생성되게 되고, 자동되는 시점은 `save()`가 호출되는 시점입니다.

`JPA` 기본 구현체에 구현되어있는 `save()` 메서드는 아래 처럼 구현되어 있습니다.

```java
@Transactional
@Override
public <S extends T> S save(S entity) {

    Assert.notNull(entity, "Entity must not be null.");

    if (entityInformation.isNew(entity)) { // (1)
        em.persist(entity);
        return entity;
    } else {
        return em.merge(entity);
    }
}
```

> (1) `isNew()`의 반환 타입에 따라 저장할지 병합할지 결정합니다.

`isNew()` 메서드는 새로운 `Entity`를 판단하기 위해 ID값을 확인하는데 기본 전략은 아래와 같슨비다.

* ID의 타입이 객체 타입(reference)일 때: `null`
* ID의 타입이 기본 타입(primitive)일 때: `0`
* `Persistable` 인터페이스를 구현한 경우: 구현체 내에서 override

```java
package org.springframework.data.domain;

import org.springframework.lang.Nullable;

public interface Persistable<ID> {

	@Nullable
	ID getId();

	/**
	 * Returns if the {@code Persistable} is new or was persisted already.
	 *
	 * @return if {@literal true} the object is new.
	 */
	boolean isNew();
}
```

`Persistable` 인터페이스는 두 가지 메서드를 가지는데 ID를 반환하는 메서드와 새로운 객체 여부를 판단하는 `isNew()` 메서드 입니다.

`save()`에서 호출했던 `isNew()`가 바로 이 메서드입니다.

따라서 `isNew()`를 `Entity`의 속성들을 이용해 원하는 방식으로 구현할 경우, 해당 로직에 의해 새로운 `Entity` 유무를 판단합니다.

다시 위의 테스트로 돌아가서 `Item` `Entity`에 `@GeneratedValue` 애너테이션이 추가되어있기 때문에 `Entity`에 `ID` 값을 따로 설정해주지 않아도 `save()` 시점에 `isNew()` 메서드를 통해 새로운 `Entity`로 판단한 뒤 생성 전략에 따라 알아서 `ID` 값을 생성해 저장해주게 됩니다.  

`ID`를 문자열 값 등으로 지정하기 위해 자동 생성 전략을 선택하지 않았을 경우 어떻게 동작할까요?

기존 `Entity`, `Repository`를 수정해보겠습니다.

```java
package io.lcalmsky.springdatajpa.domain.entity;

import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.Entity;
import javax.persistence.Id;

@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Item {
    @Id
    private String id;

    public Item(String id) {
        this.id = id;
    }
}
```

먼저 `Item`의 `ID` 필드를 `String` 타입으로 수정하였고 생성자를 통해 `ID`를 생성해서 주입하도록 수정하였습니다.

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.entity.Item;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ItemRepository extends JpaRepository<Item, String> {
}
```

`ID` 필드가 `String` 타입으로 바뀌었기 때문에 `Repository`의 `Generic` 타입도 변경해주었습니다.

```java
package io.lcalmsky.springdatajpa.domain.repository;

import io.lcalmsky.springdatajpa.domain.entity.Item;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.UUID;

@SpringBootTest
class ItemRepositoryTest {
    @Autowired
    ItemRepository itemRepository;

    @Test
    void save() {
        Item item = new Item(UUID.randomUUID().toString());
        itemRepository.save(item);
    }
}
```

`Item`의 생성자에 `UUID`를 생성해 전달하도록 수정하였습니다.

그리고 테스트를 수행해보았습니다.

테스트는 통과되었고 정상동작이 수행된 거 같지만 로그를 살펴보면

```text
2021-07-14 04:19:28.448 DEBUG 57072 --- [           main] org.hibernate.SQL                        : 
    select
        item0_.id as id1_0_0_ 
    from
        item item0_ 
    where
        item0_.id=?
2021-07-14 04:19:28.452 DEBUG 57072 --- [           main] org.hibernate.SQL                        : 
    insert 
    into
        item
        (id) 
    values
        (?)
```

`select` 이후 `insert` 하는 것을 확인할 수 있습니다.

그 이유는 앞에서 `ID` 필드에 값을 세팅해주었기 때문에 `isNew()`에서 (`ID` 필드가 `null`이 아니라) `false`를 반환하였고 그 결과 `merge()`가 호출되었기 때문입니다.

여기서 확인할 수 있는 `JPA` 기본 동작이 바로 `select` - `insert`(또는 `update`) 인데요, 이 부분을 간과하고 그냥 사용하시게되면 실무에서 정말 큰 재앙을 불러올 수 있습니다.

"없으면 추가하고 있으면 업데이트해라"는 우리가 개발을 하면서 많이 사용하는 전략인데요, 업데이트 할 때도 업데이트 해야 할 필드만 선택적으로 하는 것이 아닌 객체 전체가 바꿔지기 때문에 `merge()` 기능을 의도하고 사용해야 할 일은 거의 없다고 생각하시면 됩니다.

변경을 위해선 변경 감지(dirty-checking)를, 저장을 위해선 `persist()`만이 호출되도록 유도해야 실무에서 성능 이슈 등을 경험하지 않을 수 있습니다.

`JPA`를 사용한다는 전제하에는 `ID` 필드를 되도록이면 `Long` 타입으로 지정하고 `@GeneratedValue`를 같이 사용하신다면 위에 든 예시같은 걱정은 절대 할 일이 없겠지만, 아쉽게도 실무에서는 여러 가지 사정에 의해 `String` 타입의 `PK`를 사용하게 되는 일이 높은 확률로 발생합니다.

이럴 때 `Persistable` 인터페이스를 직접 구현하시면 됩니다.

```java
package io.lcalmsky.springdatajpa.domain.entity;

import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.domain.Persistable;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.Entity;
import javax.persistence.EntityListeners;
import javax.persistence.Id;
import java.time.LocalDateTime;

@Entity
@EntityListeners(AuditingEntityListener.class) // (1)
@Getter // (2)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Item implements Persistable<String> { // (3)
    @Id
    private String id;

    @CreatedDate
    private LocalDateTime createdDate; // (4)

    public Item(String id) {
        this.id = id;
    }

    @Override
    public boolean isNew() {
        return createdDate == null; // (5)
    }
}
```

> (1) [`Auditing` 사용](https://jaime-note.tistory.com/60)을 위해 `@EventListeners`를 추가해주었습니다.  
> (2) `@Getter`가 있기 때문에 `Persitable` 메서드 중 `getId`를 추가로 구현할 필요가 없습니다.  
> (3) `Persistable`을 구현하게 하였습니다.  
> (4) `ID` 필드 하나만으로는 새로 생성한 것인지 판단하기 어렵기 때문에 생성 날짜 필드를 추가하였습니다.  
> (5) `isNew()` 메서드를 `Override`해 생성 날짜가 없으면 새로운 `Entity`임을 판단하게 하였습니다.  

위 같이 수정하고 테스트를 다시 실행해보면,

```text
2021-07-14 04:34:52.608 DEBUG 57181 --- [           main] org.hibernate.SQL                        : 
    insert 
    into
        item
        (created_date, id) 
    values
        (?, ?)
```

`insert` 쿼리 하나만 존재하는 것을 확인할 수 있습니다.

---

실무에서 `JPA`를 처음 도입하는 시점에 정말 많이 하는 실수가 모든 `insert`, `update`에 대해 `select`를 사전에 수행하는 것인데요, 특히 배치로 파일이나 다른 DB를 읽어와 작업하는 경우, 대량의 데이터를 한 번에 `merge()`하면 메모리 관련 에러가 발생할 수도 있고, 프로세스가 죽는 등 심각한 상황을 초래할 수 있습니다. 

제가 네이버 면접을 여러 번 경험했었는데 이 부분에 대해 어떻게 처리했는지 질문을 수 차례 받았던 기억이 있습니다. 그 만큼 `JPA`를 사용하면서 반드시 알아야 할 부분이라는 반증이기도 하겠지요. 😁