스프링 데이터 JPA가 제공하는 공통 구현체를 분석해봅시다.

`SimpleJpaRepository`는 `JpaRepository`를 구현하는 구현체로 기본적인 `CRUD`가 어떻게 동작하는지 확인할 수 있습니다.

약 1000 라인에 가까운 소스 코드가 내부적으로 `EntityManager`를 주입받아 `JPA`를 직접 사용하는 방식으로 구현되어 있습니다.

몇 가지 구현된 기능들을 살펴보겠습니다.

### 조회

`CRUD`중 `R`에 해당하는 것으로 `select` 동작 중 `PK`로 한 `row`를 가져올 때 사용합니다.

```java
@Override
public Optional<T> findById(ID id) {

    Assert.notNull(id, ID_MUST_NOT_BE_NULL);

    Class<T> domainType = getDomainClass(); // (1)

    if (metadata == null) { // (2)
        return Optional.ofNullable(em.find(domainType, id)); // (3)
    }

    LockModeType type = metadata.getLockModeType(); // (4)

    Map<String, Object> hints = new HashMap<>();
    getQueryHints().withFetchGraphs(em).forEach(hints::put); // (5)

    return Optional.ofNullable(type == null ? em.find(domainType, id, hints) : em.find(domainType, id, type, hints)); // (6)
}
```

> (1) `Entity` 클래스의 타입을 가져옵니다.  
> (2) `metadata`는 `JpaRepository` 인터페이스를 상속한 인터페이스에서 정의한 애너테이션 정보를 갖고있습니다. 특별한 애너테이션이 없으면 기본 `select` 동작을 수행합니다.    
> (3) `EntityManager`의 `find` 메서드를 호출해 가장 기본 파라미터만 가지고 조회합니다.  
> (4) `metadata`에 `Lock`이 정의되어있을 경우 `LockModeType`을 획득합니다.  
> (5) `metadata`에 `Hint`가 정의되어있을 경우 정의된 `Hint` 모두를 파라미터로 전달할 수 있게 `Map`에 매핑합니다.  
> (6) 오버로딩된 `find` 메서드를 호출합니다. `type`, `lock`, `hint` 정보가 추가로 포함될 수 있으므로 해당 정보 유무에 따라 필요한 메서드를 호출합니다.

단순 `select` 하는 기능이기 때문에 복잡한 내용은 전혀 없습니다. 직접 구현하더라도 오래 걸리지 않을 정도로 단순하게 구현되어있고, 차이점이라고 하면 구현체가 생성되는 시점에 `Reflection` 등을 활용해 애너테이션을 사전에 불러오는 작업이 추가되었다고 볼 수 있습니다.

이는 `select` 관련 메서드 모두에 해당합니다. 사용자가 추가로 구현할 필요 없게 다양한 파라미터를 활용할 수 있도록 오버로딩 되어있어 파라미터나 리턴 타입 등을 유연하게 지정할 수 있습니다.

### QueryUtils

`SimpleJpaRepository`에서 사용하는 `Util` 클래스로 쿼리를 생성하기 위해 사용합니다.

내부적으로 다양한 포맷의 문자열들을 `static` 변수로 가지고 있고, 파라미터가 적절한 위치에 매핑되는 형태의 메서드도 제공합니다.

```java
public abstract class QueryUtils {

    public static final String COUNT_QUERY_STRING = "select count(%s) from %s x";
    public static final String DELETE_ALL_QUERY_STRING = "delete from %s x";
    public static final String DELETE_ALL_QUERY_BY_ID_STRING = "delete from %s x where %s in :ids";
    private static final String IDENTIFIER = "[._$[\\P{Z}&&\\P{Cc}&&\\P{Cf}&&\\P{Punct}]]+";
    static final String COLON_NO_DOUBLE_COLON = "(?<![:\\\\]):";
    static final String IDENTIFIER_GROUP = String.format("(%s)", IDENTIFIER);

    private static final String COUNT_REPLACEMENT_TEMPLATE = "select count(%s) $5$6$7";
    private static final String SIMPLE_COUNT_VALUE = "$2";
    private static final String COMPLEX_COUNT_VALUE = "$3 $6";
    private static final String COMPLEX_COUNT_LAST_VALUE = "$6";
    private static final String ORDER_BY_PART = "(?iu)\\s+order\\s+by\\s+.*";

    private static final Pattern ALIAS_MATCH;
    private static final Pattern COUNT_MATCH;
    private static final Pattern PROJECTION_CLAUSE = Pattern.compile("select\\s+(?:distinct\\s+)?(.+)\\s+from",
            Pattern.CASE_INSENSITIVE);

    private static final Pattern NO_DIGITS = Pattern.compile("\\D+");

    private static final String JOIN = "join\\s+(fetch\\s+)?" + IDENTIFIER + "\\s+(as\\s+)?" + IDENTIFIER_GROUP;
    private static final Pattern JOIN_PATTERN = Pattern.compile(JOIN, Pattern.CASE_INSENSITIVE);

    private static final String EQUALS_CONDITION_STRING = "%s.%s = :%s";
    private static final Pattern ORDER_BY = Pattern.compile(".*order\\s+by\\s+.*", CASE_INSENSITIVE);

    private static final Pattern NAMED_PARAMETER = Pattern.compile(COLON_NO_DOUBLE_COLON + IDENTIFIER + "|#" + IDENTIFIER,
            CASE_INSENSITIVE);

    private static final Pattern CONSTRUCTOR_EXPRESSION;

    private static final Map<PersistentAttributeType, Class<? extends Annotation>> ASSOCIATION_TYPES;

    private static final int QUERY_JOIN_ALIAS_GROUP_INDEX = 3;
    private static final int VARIABLE_NAME_GROUP_INDEX = 4;
    private static final int COMPLEX_COUNT_FIRST_INDEX = 3;

    private static final Pattern PUNCTATION_PATTERN = Pattern.compile(".*((?![._])[\\p{Punct}|\\s])");
    private static final Pattern FUNCTION_PATTERN;
    private static final Pattern FIELD_ALIAS_PATTERN;

    private static final String UNSAFE_PROPERTY_REFERENCE = "Sort expression '%s' must only contain property references or "
            + "aliases used in the select clause. If you really want to use something other than that for sorting, please use "
            + "JpaSort.unsafe(…)!";
    // 생략
}
```

`QueryUtils`에 정의된 상수들만 가져온 것인데 `static` 영역에서 초기화하는 부분은 제외시켰는데도 불구하고 양이 상당합니다.

쿼리의 종류가 많기 때문에 동적 쿼리를 제외한 어느 정도 정해져있는 쿼리들만 표현하더라도 당연히 상당한 양이겠지요.

단순 포맷 문자열도 있고 패턴 매칭을 위한 문자열도 존재함을 알 수 있습니다.

그 밖에 쿼리를 만들기 위한 `build` 메서드들, 정렬 옵션등을 적용하기 위한 메서드 등 다양한 기능을 제공합니다.

### 애너테이션

`SimpleJpaRepository`에서 기본적으로 제공하는 애너테이션을 살펴보면,

```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
    // 생략
}
```

이렇게 두 가지 입니다.

`@Repository` 애너테이션이 제공하는 기능은 스프링에게 `Component`임을 알려 `ComponentScan` 대상이 되는 것 뿐만 아니라 DB와 연동하면서 발생하는 다양한 에러들을 스프링 에러로 모두 바꿔주는 역할을 합니다.

DB 연동 방식을 `JdbcTemplate`으로 바꾸거나 다른 프레임워크를 이용하더라도 `Repository` 애너테이션이 추가되어있다면 추상화된 예외처리 방식으로 구현 가능하고, 이는 실제 구현체가 바뀌어도 다른 비즈니스 로직에는 영향을 주지 않는 객체지향적인 설계가 반영된 것입니다.

`Repository` 레이어 호출 전까지 `Transaction`을 제어할 수 있는 애너테이션을 사용하지 않았다면 `Repository` 레이어에서 기본적으로 `readOnly = true` 옵션을 사용하여 동작합니다.

`SimpleJpaRepository` 내에서는 조회 기능이 압도적으로 많기 때문에 기본 동작이 `readOnly` 이고, `save` 등을 호출하는 메서드에는 `@Transactional` 애너테이션이 다시 추가되어있어 어떻게 처리할지에 대한 내용을 `override` 합니다.

### persist or merge

`save` 메서드의 기본 동작은 저장(save) 또는 합병(merge)이고 구현 내용은 아래와 같습니다. 

```java
@Transactional
@Override
public <S extends T> S save(S entity) {

    Assert.notNull(entity, "Entity must not be null.");

    if (entityInformation.isNew(entity)) { // (1)
        em.persist(entity);
        return entity;
    } else { // (2)
        return em.merge(entity);
    }
}
```
> (1) 새로운 `Entity`인지 판별해 영속성 컨텍스트로 저장합니다.  
> (2) 기존 `Entity`라고 판단하였으면 영속성 컨텍스트에 `merge` 합니다.

이러한 구현이 처음 `JPA`를 개발하는 분들에게 상당히 많은 혼란을 가져다주고 성능 이슈를 유발하거나 기대했던 동작과 다른 결과를 얻게하는 주범이기도 한데요, 업데이트는 `merge`가 아닌 변경 감지(dirty-checking)를 통해 이루어져야 합니다. `merge`가 발생하면 해당 객체를 완전히 교체해 DB에 업데이트 하기 때문입니다.

새로운 `Entity`인지 판단하는 기준은 영속성 컨텍스트에 저장되어있는지 여부입니다. 영속성 컨텍스트에 저장되어있다는 건 DB에 한 번 저장되었다는 뜻이기도 하죠.

이 부분을 `JPA`에선 어떻게 처리하는지는 다음 포스팅에서 다룰 예정입니다.