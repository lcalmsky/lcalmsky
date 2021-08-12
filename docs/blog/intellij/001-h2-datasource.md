로컬 환경에서 데이터 베이스를 설정할 때 `H2 Database`를 많이 사용하고 특히 메모리 기반으로 동작시키는 상황이 자주 발생합니다.

`IntelliJ IDEA` 에서 `Database` 탭을 활용하면 따로 접속 툴을 이용할 필요가 없어 굉장히 편리합니다.

보통 로컬에 `MySQL Workbench`나 `H2 Database`를 따로 설치해서 실행시킨 뒤 `Database` 탭 내에서 `DataSource`를 추가하여 로컬DB와 연동하는 방식을 많이 사용하는데요, 오늘 소개할 방법은 로컬에 따로 `Database`를 띄우지 않고 메모리 기반의 `DataSource`를 추가하는 방법입니다.

---

> 결과물을 보여드리기 위해 스프링 부트 프로젝트를 만들어 작성하였습니다. 설정하는 방법만 필요하신 분은 [여기](#DataSource-설정) 참고하시면 됩니다.  
> 소스 코드는 [여기](https://github.com/lcalmsky/in-memory-h2-database) 있습니다.

### 프로젝트 생성 및 소스 코드 작성

먼저 `H2 Database` 사용을 위해선 `dependency` 추가가 필요합니다.

* build.gradle
```groovy
dependencies {
    // 생략
    implementation 'org.springframework.boot:spring-boot-starter-web' // (1)
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa' // (2)
    runtimeOnly 'com.h2database:h2' // (3)
    annotationProcessor 'org.projectlombok:lombok' // (4)
    // 생략
}
```

> (1) 서버를 띄우기 위해 추가하였습니다.  
> (2), (4) 나중에 결과 확인을 위해 `Entity`를 작성하려고 추가하였습니다.  
> (3) 실제로 운영에서 `H2 Database`를 사용하는 경우는 거의 없기 때문에 로컬에서 테스트 용으로만 사용할 수 있게 `runtimeOnly`로 추가합니다.

다음으로 설정을 추가합니다.

* application-local.yml
```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb # DB 접속 URL
    username: sa # DB 접속 username
    password: # DB 접속 password, 없음
    driver-class-name: org.h2.Driver # DB 접속 드라이버
  h2.console:
    enabled: true # 콘솔 사용 여부, 로컬에서 웹 브라우저로도 접속 가능
  jpa:
    hibernate:
      ddl-auto: create # 테이블을 자동으로 생성함
    properties:
      hibernate:
        format_sql: true # SQL 로그 출력
logging:
  level:
    org.hibernate.SQL: debug # SQL 로그 출력을 위한 로그 레벨 조정
```

대부분 기본 값이지만 직관적으로 확인할 수 있도록 작성하였습니다.

그리고 테이블 자동 생성을 위해 간단히 `Entity` 두 개만 작성해보겠습니다.

```java
package io.lcalmsky.inmemoryh2database.domain.entity;

import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.ToString;

import javax.persistence.*;

@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@ToString
public class Player {
    @Id
    @GeneratedValue
    @Column(name = "player_id")
    private Long id;
    private Integer number;
    private String name;
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private Team team;
}
```

```java
package io.lcalmsky.inmemoryh2database.domain.entity;

import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.ToString;

import javax.persistence.*;
import java.util.List;

@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@ToString
public class Team {
    @Id
    @GeneratedValue
    @Column(name = "team_id")
    private Long id;
    private String name;
    @OneToMany(mappedBy = "team")
    @ToString.Exclude
    private List<Player> players;
}
```

여기까지 작성을 완료했으면 한 번 실행해볼까요? (실행 전에 `active profile`을 `local`로 설정해주는 것 잊지마세요!)

```text
2021-08-12 11:21:01.868  INFO 2520 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
2021-08-12 11:21:02.155 DEBUG 2520 --- [           main] org.hibernate.SQL                        : 
    
    drop table if exists player CASCADE 
2021-08-12 11:21:02.156 DEBUG 2520 --- [           main] org.hibernate.SQL                        : 
    
    drop table if exists team CASCADE 
2021-08-12 11:21:02.157 DEBUG 2520 --- [           main] org.hibernate.SQL                        : 
    
    drop sequence if exists hibernate_sequence
2021-08-12 11:21:02.158 DEBUG 2520 --- [           main] org.hibernate.SQL                        : create sequence hibernate_sequence start with 1 increment by 1
2021-08-12 11:21:02.160 DEBUG 2520 --- [           main] org.hibernate.SQL                        : 
    
    create table player (
       player_id bigint not null,
        name varchar(255),
        number integer,
        team_id bigint,
        primary key (player_id)
    )
2021-08-12 11:21:02.163 DEBUG 2520 --- [           main] org.hibernate.SQL                        : 
    
    create table team (
       team_id bigint not null,
        name varchar(255),
        primary key (team_id)
    )
2021-08-12 11:21:02.164 DEBUG 2520 --- [           main] org.hibernate.SQL                        : 
    
    alter table player 
       add constraint FKdvd6ljes11r44igawmpm1mc5s 
       foreign key (team_id) 
       references team
2021-08-12 11:21:02.172  INFO 2520 --- [           main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
2021-08-12 11:21:02.176  INFO 2520 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2021-08-12 11:21:02.222  WARN 2520 --- [           main] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
2021-08-12 11:21:02.527  INFO 2520 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-08-12 11:21:02.542  INFO 2520 --- [           main] i.l.i.InMemoryH2DatabaseApplication      : Started InMemoryH2DatabaseApplication in 2.906 seconds (JVM running for 3.546)
```

이렇게 메모리 기반 DB에 테이블이 존재하면 삭제한 뒤 다시 생성(spring.jpa.hibernate.ddl-auto=create)하는 것을 확인할 수 있습니다.

### DataSource 설정

다음으로 `IntelliJ IDEA` 내에서 `DataSource`를 추가해보겠습니다.

맥 기준 `CMD` + `Shift` + `A`를 누른 뒤 검색 화면에 `Database`를 입력하면 `Database` 창이 우측에 표시됩니다.

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-001-01.png)

여러 가지 방법으로 `DataSource`를 추가할 수 있으나 가장 편리한 방법은 바로 `URL`을 복사하는 것 인데요, 위에 설정에 입력했던 URL 정보(spring.datasource.url=jdbc:h2:mem:testdb)를 복사한 뒤 `Database` 창에서 더하기 버튼 클릭 후 `Data Source from URL`을 클릭한 뒤 `URL`을 입력합니다.  

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-001-02.png)

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-001-03.png)

그럼 다음과 같은 창이 나타나는데 `username`만 설정에 적용한 대로 `sa`로 입력해주시면 됩니다.

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-001-04.png)

그리고 `OK` 버튼을 누르시면 `Database` 창 내에 `DataSource`가 추가되고 바로 에디터 창에는 `SQL Console`이 띄워져 그 곳에서 직접 `SQL`문을 실행할 수 있습니다.

---

스프링 부트 애플리케이션을 실행하면 방금 추가한 `DataSource`에 테이블이 생성될 줄 알았는데요, 실제 동작할 때는 매 번 `JVM`내 새로운 메모리에 할당한다고 `H2` 공식 문서에 나와있네요😭

이 `DataSource`를 활용할 수 있는 방법은 스프링 부트 애플리케이션을 실행했을 때 로그로 출력되는 테이블 추가하는 `SQL`문을 `SQL Console`에 입력하여 다시 확인해보는 정도가 될 거 같습니다.

결과적으로 정리하자면 `remote`나 `file` DB를 사용하지 않는 이상 `DataSource`로 지정하더라도 생성한 테이블이나 데이터들이 남지 않게 됩니다.

---

그럼 로컬에 `H2 Database`를 따로 띄우지 않고 `file`을 이용한 방법으로 다시 한 번 만들어보겠습니다. (여기까지 작성한 게 아까워서라도 뭐라도 건지고자..)

먼저 설정을 파일을 볼 수 있도록 바꿔줍니다.

* application-local.yml
```yaml
spring:
  datasource:
    url: jdbc:h2:file:./testdb
  # 생략
```

`file:` 뒤에 나오는 값은 `/`로 시작하면 절대경로, `.`으로 시작하면 상대경로로 동작합니다.

즉 `./testdb` 라는 것은 **프로젝트 루트 폴더 내에 testdb라는 파일을 저장소로 사용**하겠다는 뜻 입니다.

그리고 위에서 추가한 방식과 동일하게 URL로 DataSource를 추가해줍니다.

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-001-05.png)

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-001-06.png)

캡쳐를 보시면 아시겠지만 `connection type`이 `embedded`로 바뀌었습니다. `username`만 따로 세팅 후 확인을 눌러 `DataSource` 생성을 마친 뒤 다시 애플리케이션을 실행해보겠습니다.

애플리케이션을 실행한 뒤 `DataSource`를 보면 아무런 변화가 없는데요, 뭐가 잘못 입력됐나 싶어 `Data Source Properties`를 다시 열어 `Test Connection`을 확인해보니..

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-001-07.png)

```text
Database may be already in use: null
The file is locked: nio:/Users/jaime/git-repo/in-memory-h2-database/testdb.mv.db
```

이런 에러 문구가 노출됐는데요, 데이터베이스가 이미 사용중이고 파일에 lock이 걸려있다는 내용입니다.

이미 애플리케이션을 실행중이었기에 애플리케이션 프로세스를 중지시키고 다시 확인해보면 정상적으로 테이블 두 개가 생성된 걸 확인할 수 있습니다.

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-001-08.png)

매 번 이렇게 사용할 순 없기에 파일에 `lock 모드`를 사용하지 않게 바꿔주기만 하면 모든 게 해결될 거 같습니다.

`application-local.yml` 파일과 `Data Source Properties`의 `URL`을 모두 아래 처럼 변경해줍니다.

* application-local.yml
```yaml
spring:
  datasource:
    url: jdbc:h2:file:./testdb;DB_CLOSE_ON_EXIT=FALSE;AUTO_SERVER=TRUE
  # 생략
```

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-001-09.png)

그리고 애플리케이션을 재실행 한 뒤 `DataSource`에서 `Test Connection`을 실행해보면 정상적으로 수행되는 것을 확인할 수 있습니다.

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-001-10.png)

이제 동기화가 되었으니 `h2-console`을 따로 접속할 필요 없이 `IntelliJ IDEA` 내에서 편리하게 `DB`를 확인할 수 있겠네요!

---

추가로 `DataSource`가 정상적으로 추가 및 연결되었다면 `ER Diagram`을 확인할 수 있는데요(사실 이 기능을 사용하려고 여태까지 이 짓들을..),

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-001-11.png)

캡쳐에 나온대로 사용할 `Database` 내에 `tables` 디렉토리(또는 직접 테이블을 선택해도 됨)를 우클릭 후 `Diagrams` - `Show Visualization`을 선택하면

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-001-12.png)

이렇게 `ER Diagram`을 확인할 수 있습니다. 👏