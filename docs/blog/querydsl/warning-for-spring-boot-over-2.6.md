스프링 부트 2.6 버전 이상부터 Querydsl 5.0을 기본으로 지원하게 되면서 몇 가지 주의해야할 사항들이 있습니다.

## PageableExecutionUtils 패키지 변경

먼저 `PageableExecutionUtils`의 패키지 위치가 변경되었습니다.

기존 위치는 `org.springframework.data.repository.support.PageableExecutionUtils`였는데 `org.springframework.data.support.PageableExecutionUtils`로 변경되었습니다.

기존에 사용하고 있었다고 하면 `import`를 다시 해주셔야 합니다.

## Querydsl fetchResults(), fetchCount() Deprecated

`Querydsl`의 `fetchCount()`와 `fetchResult()`는 `select` 쿼리를 기반으로 `count` 쿼리를 내부에서 만들어서 실행하는데, 이 기능 자체는 단순히 `select`로 `count`를 처리하는 용도였습니다.

따라서 단순한 쿼리에서는 정상동작하였지만 쿼리가 조금만 복잡해져도 기대한 결과를 얻기 어려웠습니다.

그래서 해당 기능들은 모두 `deprecated` 되었습니다.

하지만 `deprecated`이기 때문에 아직은(?) 정상 동작하는 코드를 수정할 필요는 없습니다만 warning이 많이 거슬리시는 분들은 아래 처럼 `count` 쿼리를 수정해주셔야 합니다.

```sql
select count(member.id) from member
```

```java
Long totalCount = queryFactory
  .select(member.count())
  .from(member)
  .fetchOne();
```


```sql
select count(*) from member
```

```java
Long totalCount = queryFactory
  .select(Wildcard.count)
  .from(member)
  .fetchOne();
```

이런식으로 사용 가능하고 `count`의 결과가 한 개이기 때문에 `fetchOne()`을 사용해야 합니다.