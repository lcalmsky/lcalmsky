![](https://img.shields.io/badge/spring--boot-2.5.1-red) ![](https://img.shields.io/badge/gradle-7.0.2-brightgreen)

> 모든 소스 코드는 [여기](https://github.com/lcalmsky/spring-data-jpa) 에서 확인 가능합니다.

### API에서 페이징 활용

지난 번에 스프링 데이터 `JPA`가 페이징 및 정렬을 지원하는 부분을 살펴봤었는데요, API 형태로 제공될 때는 어떻게 활용될 수 있는지 `Controller`를 개발해 확인해보겠습니다.

```java
package io.lcalmsky.springdatajpa.controller;

import io.lcalmsky.springdatajpa.domain.entity.Member;
import io.lcalmsky.springdatajpa.domain.repository.MemberRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.PostConstruct;
import java.util.stream.IntStream;

@RestController
@RequiredArgsConstructor
public class MemberController {
    private final MemberRepository memberRepository; // (1)

    @GetMapping("/members")
    public Page<Member> list(Pageable pageable) { // (2)
        return memberRepository.findAll(pageable); // (3)
    }

    @PostConstruct
    public void init() { // (4)
        IntStream.range(1, 100).forEach(i -> memberRepository.save(new Member("member" + i, (int) (Math.random() * i))));
    }
}
```

> (1) 기능 확인을 위해 바로 `MemberRepository` 의존성을 주입합니다.  
> (2) `Pageable` 인터페이스를 파라미터로 지정합니다. 스프링은 페이징 관련 쿼리 파라미터를 `Pageable` 구현체인 `PageRequest`로 매핑하여 전달합니다.  
> (3) `findAll(Pageable pageable)` 메서드를 호출합니다. `Pageable`을 파라미터로 받는 `findAll` 메서드는 `PagingAndSortRepository` 인터페이스에 정의되어있습니다. 반환 타입이 `Page<T>` 이기 때문에 `totalCount`를 구하기 위한 쿼리가 한 번 추가됩니다.  
> (4) 테스트를 위해 애플리케이션 시작 후 `Controller` 빈이 등록될 때 100개의 데이터를 `MemberRepository`에 저장합니다.

이 상태에서 애플리케이션을 실행한 뒤 웹 브라우저에서 `http://localhost:8080/members`를 입력하거나 클라이언트 툴 또는 `CURL`을 이용해 요청해보면 다음과 같은 응답을 얻을 수 있습니다.

* CURL 사용
```curl
curl -X GET --location "http://localhost:8080/members"
```

* HTTP 사용
```http request
GET http://localhost:8080/members
```

* 결과
```text
{
  "content": [
    {
      "createdDate": "2021-07-12T02:21:38.965867",
      "lastModifiedDate": "2021-07-12T02:21:38.965867",
      "createdBy": "a6a4b703-0a48-4265-ab65-476418b56eb4",
      "lastModifiedBy": "a6a4b703-0a48-4265-ab65-476418b56eb4",
      "id": 1,
      "username": "member1",
      "age": 0,
      "team": null
    },
    // 생략
    {
      "createdDate": "2021-07-12T02:21:39.03094",
      "lastModifiedDate": "2021-07-12T02:21:39.03094",
      "createdBy": "b1d8dad7-f031-4e5d-8861-f78d777847dc",
      "lastModifiedBy": "b1d8dad7-f031-4e5d-8861-f78d777847dc",
      "id": 20,
      "username": "member20",
      "age": 17,
      "team": null
    }
  ],
  "pageable": {
    "sort": {
      "unsorted": true,
      "sorted": false,
      "empty": true
    },
    "offset": 0,
    "pageNumber": 0,
    "pageSize": 20,
    "paged": true,
    "unpaged": false
  },
  "totalPages": 5,
  "totalElements": 99,
  "last": false,
  "sort": {
    "unsorted": true,
    "sorted": false,
    "empty": true
  },
  "number": 0,
  "size": 20,
  "numberOfElements": 20,
  "first": true,
  "empty": false
}
```

아무 것도 구현하지 않고 아무 것도 세팅하지 않은 채로 요청했지만 `Member` 데이터 외에도 페이징을 위해 사용할 수 있는 값들이 반환됩니다.

응답 전문을 유심히 보신 분들은 눈치채셨겠지만 파라미터로 아무 것도 설정하지 않았을 때 페이지의 기본 사이즈는 `20`입니다. 그 외에 첫 페이지인 `0`부터 요청하는 것을 알 수 있고 정렬 옵션은 아무것도 지정하지 않는 것을 확인할 수 있습니다.

클라이언트에서 페이징 관련 옵션을 지정하기 위해선 쿼리 파라미터를 사용하는데 지원하는 쿼리 파라미터는 다음과 같습니다.

* page: 현재 페이지, 0부터 시작
* size: 한 페이지에 노출될 데이터(기본 20)
* sort: 정렬 조건, 필드, 오름/내림차순 순으로 입력, 기본은 오름차순

세 번 째 페이지를 조회하면서 페이지당 5개의 데이터를 노출하고 ID를 역순으로 출력하기 위해선

```curl
curl -X GET --location "http://localhost:8080/members?page=3&size=5&sort=id,desc"
```

```http request
GET http://localhost:8080/members?page=3&size=5&sort=id,desc
```

이런식으로 요청하면 됩니다.

### 기본 설정 지원

바로 위에 페이지 사이즈를 따로 지정하지 않았을 때 기본 사이즈가 20이라고 설명드렸는데요, 이 기본 값을 스프링 설정을 이용해 변경할 수 있습니다.

```properties
spring.data.web.pageable.default-page-size: 30
spring.data.web.pageable.max-page-size: 3000
```

```yaml
spring:
  data:
    web:
      pageable:
        default-page-size: 30
        max-page-size: 3000
```

이렇게 기본 페이지 사이즈와 최대 페이지 사이즈를 지정할 수 있습니다. 최대 페이지 사이즈는 클라이언트에서 더 큰 수를 입력하더라도 지정한 값의 데이터만 반환하도록 강제할 수 있습니다.

### 개별 설정 지원

위에서 기본 설정을 글로벌하게 지정했다면, API 별로 설정할 수도 있습니다. `@PageableDefault` 애너테이션의 속성을 통해 지정 가능합니다.

```java
@GetMapping("/members")
public Page<Member> list(@PageableDefault(size = 10, sort = {"username"}, direction = Sort.Direction.DESC) Pageable pageable) {
    return memberRepository.findAll(pageable);
}
```

사이즈를 `10`으로, 정렬은 `username` 기준 `역순`으로 지정하였습니다.

이 상태에서 애플리케이션을 실행하고 다시 아무런 쿼리 파라미터 없이 요청을 해보면,

```text
{
  "content": [
    {
      "createdDate": "2021-07-12T03:02:33.473022",
      "lastModifiedDate": "2021-07-12T03:02:33.473022",
      "createdBy": "37ae2679-b208-497a-8831-9a10247a0d23",
      "lastModifiedBy": "37ae2679-b208-497a-8831-9a10247a0d23",
      "id": 99,
      "username": "member99",
      "age": 71,
      "team": null
    },
    // 생략
    {
      "createdDate": "2021-07-12T03:02:33.460063",
      "lastModifiedDate": "2021-07-12T03:02:33.460063",
      "createdBy": "c8bc8070-3a95-4f1e-8a83-21944d667e45",
      "lastModifiedBy": "c8bc8070-3a95-4f1e-8a83-21944d667e45",
      "id": 90,
      "username": "member90",
      "age": 20,
      "team": null
    }
  ],
  "pageable": {
    "sort": {
      "sorted": true,
      "unsorted": false,
      "empty": false
    },
    "pageSize": 10,
    "pageNumber": 0,
    "offset": 0,
    "paged": true,
    "unpaged": false
  },
  "totalPages": 10,
  "totalElements": 99,
  "last": false,
  "sort": {
    "sorted": true,
    "unsorted": false,
    "empty": false
  },
  "size": 10,
  "numberOfElements": 10,
  "number": 0,
  "first": true,
  "empty": false
}
```

`member99`부터 `10`개의 데이터가 노출되는 것을 확인할 수 있습니다.

### 접두사

하나의 API에 두 개 이상의 `Pageable` 인터페이스를 전달하기 위해서 접두사를 지정할 수 있는 `@Qualifier` 애너테이션을 사용합니다.

```java
public Something list(@Qualifier("member") Pageable memberPageable, @Qualifier("team") Pageable teamPageable) {
    // 생략
}
```

클라이언트도 마찬가지로 접두사를 붙여서 요청하면 구별된 `Pageable` 구현체를 전달할 수 있습니다.

```http request
GET http://localhost:8080/members?member_page=5&member_size=10&team_page=1&team_size=3
```

---

위의 예시는 단순히 `Pageable` 인터페이스의 구현체가 서버/클라이언트에서 어떻게 매핑되고 활용되는지 설명하기 위한 것으로, 실무에서는 `Entity`를 `API` 응답으로 바로 노출시키는 경우는 거의 없습니다.

[이전 포스팅](https://jaime-note.tistory.com/52) 에서 설명한대로 `Page`를 반환받은 뒤 반드시 `map()` 메서드를 활용하여 응답 객체에 매핑해서 반환할 수 있도록 설계 & 개발해야 합니다.