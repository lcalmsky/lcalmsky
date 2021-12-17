## Overview

터미널 명령어 중 텍스트 처리에 관련된 명령어를 정리합니다. 다양한 옵션이 존재하지만 실무에서 사용하는 것들 위주로 정리하였습니다.

## head

### Description

파일의 앞 부분을 출력합니다.

### Options

자주 사용하는 옵션을 소개합니다.

* -c, --bytes=[-]NUM: NUM byte만 출력
* -n, --lines=[-]NUM: NUM line만 출력

> NUM 에는 Kilo Bytes(K), Mega Bytes(M), Giga Bytes(G), Tera Bytes(T)를 입력할 수 있습니다. (e.g. 5K, 10M, 15G)  
> NUM앞에 -를 붙이면 마지막 NUM 만큼의 bytes 또는 lines만큼을 제외하고 출력합니다.

### Examples

`spring.log`라는 파일에 `head` 명령어를 사용해 출력해보겠습니다.

<details>
<summary>
전체 로그 파일 보기
</summary>

```text
2021-12-18 02:34:47.953  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : Starting App using Java 11.0.11 on Jaime-iMac.local with PID 46077 (/Users/jaime/git-repo/spring-boot-app/out/production/classes started by jaime in /Users/jaime/git-repo/spring-boot-app)
2021-12-18 02:34:47.957  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : The following profiles are active: local
2021-12-18 02:34:48.018  INFO 46077 --- [restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : Devtools property defaults active! Set 'spring.devtools.add-properties' to 'false' to disable
2021-12-18 02:34:48.019  INFO 46077 --- [restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : For additional web related logging consider setting the 'logging.level.web' property to 'DEBUG'
2021-12-18 02:34:49.148  INFO 46077 --- [restartedMain] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFAULT mode.
2021-12-18 02:34:49.205  INFO 46077 --- [restartedMain] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 53 ms. Found 1 JPA repository interfaces.
2021-12-18 02:34:49.850  INFO 46077 --- [restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2021-12-18 02:34:49.857  INFO 46077 --- [restartedMain] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-12-18 02:34:49.857  INFO 46077 --- [restartedMain] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.52]
2021-12-18 02:34:49.918  INFO 46077 --- [restartedMain] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-12-18 02:34:49.918  INFO 46077 --- [restartedMain] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1898 ms
2021-12-18 02:34:49.995  INFO 46077 --- [restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2021-12-18 02:34:52.348  INFO 46077 --- [restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2021-12-18 02:34:52.353  INFO 46077 --- [restartedMain] o.s.b.a.h2.H2ConsoleAutoConfiguration    : H2 console available at '/h2-console'. Database available at 'jdbc:h2:file:./testdb'
2021-12-18 02:34:52.486  INFO 46077 --- [restartedMain] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
2021-12-18 02:34:52.509  INFO 46077 --- [restartedMain] org.hibernate.Version                    : HHH000412: Hibernate ORM core version 5.4.32.Final
2021-12-18 02:34:52.578  INFO 46077 --- [restartedMain] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.1.2.Final}
2021-12-18 02:34:52.637  INFO 46077 --- [restartedMain] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
2021-12-18 02:34:53.137  INFO 46077 --- [restartedMain] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
2021-12-18 02:34:53.142  INFO 46077 --- [restartedMain] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2021-12-18 02:34:53.575  WARN 46077 --- [restartedMain] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
2021-12-18 02:34:53.708  INFO 46077 --- [restartedMain] .s.s.UserDetailsServiceAutoConfiguration : 

Using generated security password: 9c32255b-378b-426f-b121-a707a723c2c2

2021-12-18 02:34:53.781  INFO 46077 --- [restartedMain] o.s.b.a.w.s.WelcomePageHandlerMapping    : Adding welcome page template: index
2021-12-18 02:34:53.947  INFO 46077 --- [restartedMain] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 1 endpoint(s) beneath base path '/actuator'
2021-12-18 02:34:53.965  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure org.springframework.boot.autoconfigure.security.servlet.StaticResourceRequest$StaticResourceRequestMatcher@33f240e4 with []
2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Mvc [pattern='/node_modules/**'] with []
2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Mvc [pattern='/images/**'] with []
2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Ant [pattern='/h2-console/**'] with []
2021-12-18 02:34:53.991  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure any request with [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@6117e470, org.springframework.security.web.context.SecurityContextPersistenceFilter@12046c24, org.springframework.security.web.header.HeaderWriterFilter@11aa1ef1, org.springframework.security.web.csrf.CsrfFilter@2e02b351, org.springframework.security.web.authentication.logout.LogoutFilter@7a92e469, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@70debf5b, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@4b4e2362, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@427b3465, org.springframework.security.web.session.SessionManagementFilter@77e75116, org.springframework.security.web.access.ExceptionTranslationFilter@14c7b847, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@2406e5cf]
2021-12-18 02:34:54.289  INFO 46077 --- [restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2021-12-18 02:34:54.330  INFO 46077 --- [restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-12-18 02:34:54.357  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : Started App in 7.003 seconds (JVM running for 7.844)
2021-12-18 02:35:00.566  INFO 46077 --- [SpringApplicationShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2021-12-18 02:35:00.572  INFO 46077 --- [SpringApplicationShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2021-12-18 02:35:00.881  INFO 46077 --- [SpringApplicationShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.

```

</details>

#### `head spring.log`

```text
> head spring.log
2021-12-18 02:34:47.953  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : Starting App using Java 11.0.11 on Jaime-iMac.local with PID 46077 (/Users/jaime/git-repo/spring-boot-app/out/production/classes started by jaime in /Users/jaime/git-repo/spring-boot-app)
2021-12-18 02:34:47.957  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : The following profiles are active: local
2021-12-18 02:34:48.018  INFO 46077 --- [restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : Devtools property defaults active! Set 'spring.devtools.add-properties' to 'false' to disable
2021-12-18 02:34:48.019  INFO 46077 --- [restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : For additional web related logging consider setting the 'logging.level.web' property to 'DEBUG'
2021-12-18 02:34:49.148  INFO 46077 --- [restartedMain] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFAULT mode.
2021-12-18 02:34:49.205  INFO 46077 --- [restartedMain] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 53 ms. Found 1 JPA repository interfaces.
2021-12-18 02:34:49.850  INFO 46077 --- [restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2021-12-18 02:34:49.857  INFO 46077 --- [restartedMain] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-12-18 02:34:49.857  INFO 46077 --- [restartedMain] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.52]
2021-12-18 02:34:49.918  INFO 46077 --- [restartedMain] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
```

아무 옵션도 주지 않았을 경우 앞 부분부터 10줄이 출력 됩니다.

#### `head -n 5 spring.log`

```text
> head -n 5 spring.log
2021-12-18 02:34:47.953  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : Starting App using Java 11.0.11 on Jaime-iMac.local with PID 46077 (/Users/jaime/git-repo/spring-boot-app/out/production/classes started by jaime in /Users/jaime/git-repo/spring-boot-app)
2021-12-18 02:34:47.957  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : The following profiles are active: local
2021-12-18 02:34:48.018  INFO 46077 --- [restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : Devtools property defaults active! Set 'spring.devtools.add-properties' to 'false' to disable
2021-12-18 02:34:48.019  INFO 46077 --- [restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : For additional web related logging consider setting the 'logging.level.web' property to 'DEBUG'
2021-12-18 02:34:49.148  INFO 46077 --- [restartedMain] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFAULT mode.
```

`-n` 옵션을 주어 앞에서부터 5줄만 출력하였습니다.

#### `head -c 100 spring.log`

```text
> head -c 100 spring.log
2021-12-18 02:34:47.953  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : S%
```

`-c` 옵션을 주어 앞에서부터 100 bytes만 출력하였습니다.

## tail

### Description

파일의 뒷부분을 출력합니다.

### Options

자주 사용하는 옵션을 소개합니다.

* -c, --bytes=[-]NUM: NUM byte만 출력
* -n, --lines=[-]NUM: NUM line만 출력
* **-f, --follow[={name|descr}]: 추가되는 내용을 출력**
* **-F: 파일이 truncate되는 경우 다시 열어 follow(logrotate되는 파일에 사용)**

-f와 -F는 주로 실시간으로 로그를 출력할 때 사용하므로 반드시 알아두어야 할 옵션입니다.

### Examples

`spring.log`라는 파일에 `tail` 명령어를 사용해 출력해보겠습니다.

#### `tail spring.log`

```text
> tail spring.log
2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Mvc [pattern='/node_modules/**'] with []
2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Mvc [pattern='/images/**'] with []
2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Ant [pattern='/h2-console/**'] with []
2021-12-18 02:34:53.991  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure any request with [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@6117e470, org.springframework.security.web.context.SecurityContextPersistenceFilter@12046c24, org.springframework.security.web.header.HeaderWriterFilter@11aa1ef1, org.springframework.security.web.csrf.CsrfFilter@2e02b351, org.springframework.security.web.authentication.logout.LogoutFilter@7a92e469, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@70debf5b, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@4b4e2362, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@427b3465, org.springframework.security.web.session.SessionManagementFilter@77e75116, org.springframework.security.web.access.ExceptionTranslationFilter@14c7b847, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@2406e5cf]
2021-12-18 02:34:54.289  INFO 46077 --- [restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2021-12-18 02:34:54.330  INFO 46077 --- [restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-12-18 02:34:54.357  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : Started App in 7.003 seconds (JVM running for 7.844)
2021-12-18 02:35:00.566  INFO 46077 --- [SpringApplicationShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2021-12-18 02:35:00.572  INFO 46077 --- [SpringApplicationShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2021-12-18 02:35:00.881  INFO 46077 --- [SpringApplicationShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
```

아무 옵션도 주지 않았을 경우 아래 파일의 마지막 10줄이 출력됩니다.

#### `tail -n 5 spring.log`

```text
> tail -n 5 spring.log
2021-12-18 02:34:54.330  INFO 46077 --- [restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-12-18 02:34:54.357  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : Started App in 7.003 seconds (JVM running for 7.844)
2021-12-18 02:35:00.566  INFO 46077 --- [SpringApplicationShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2021-12-18 02:35:00.572  INFO 46077 --- [SpringApplicationShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2021-12-18 02:35:00.881  INFO 46077 --- [SpringApplicationShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
```

#### `-f`, `-F`

위 옵션은 실시간으로 실행되는 애플리케이션 로그를 follow 할 때 사용하므로 적절한 예시를 넣기가 어렵네요.

먼저 `tail -f spring.log` 명령어를 입력한 뒤

```text
> tail -f spring.log
2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Mvc [pattern='/node_modules/**'] with []
2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Mvc [pattern='/images/**'] with []
2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Ant [pattern='/h2-console/**'] with []
2021-12-18 02:34:53.991  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure any request with [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@6117e470, org.springframework.security.web.context.SecurityContextPersistenceFilter@12046c24, org.springframework.security.web.header.HeaderWriterFilter@11aa1ef1, org.springframework.security.web.csrf.CsrfFilter@2e02b351, org.springframework.security.web.authentication.logout.LogoutFilter@7a92e469, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@70debf5b, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@4b4e2362, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@427b3465, org.springframework.security.web.session.SessionManagementFilter@77e75116, org.springframework.security.web.access.ExceptionTranslationFilter@14c7b847, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@2406e5cf]
2021-12-18 02:34:54.289  INFO 46077 --- [restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2021-12-18 02:34:54.330  INFO 46077 --- [restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-12-18 02:34:54.357  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : Started App in 7.003 seconds (JVM running for 7.844)
2021-12-18 02:35:00.566  INFO 46077 --- [SpringApplicationShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2021-12-18 02:35:00.572  INFO 46077 --- [SpringApplicationShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2021-12-18 02:35:00.881  INFO 46077 --- [SpringApplicationShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
```

spring.log 파일 마지막에 다섯 줄을 추가해보겠습니다.

```text
2021-12-18 02:34:53.781  INFO 46077 --- [restartedMain] o.s.b.a.w.s.WelcomePageHandlerMapping    : Adding welcome page template: index
2021-12-18 02:34:53.947  INFO 46077 --- [restartedMain] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 1 endpoint(s) beneath base path '/actuator'
2021-12-18 02:34:53.965  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure org.springframework.boot.autoconfigure.security.servlet.StaticResourceRequest$StaticResourceRequestMatcher@33f240e4 with []
2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Mvc [pattern='/node_modules/**'] with []
2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Mvc [pattern='/images/**'] with []
2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Ant [pattern='/h2-console/**'] with []
2021-12-18 02:34:53.991  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure any request with [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@6117e470, org.springframework.security.web.context.SecurityContextPersistenceFilter@12046c24, org.springframework.security.web.header.HeaderWriterFilter@11aa1ef1, org.springframework.security.web.csrf.CsrfFilter@2e02b351, org.springframework.security.web.authentication.logout.LogoutFilter@7a92e469, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@70debf5b, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@4b4e2362, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@427b3465, org.springframework.security.web.session.SessionManagementFilter@77e75116, org.springframework.security.web.access.ExceptionTranslationFilter@14c7b847, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@2406e5cf]
2021-12-18 02:34:54.289  INFO 46077 --- [restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2021-12-18 02:34:54.330  INFO 46077 --- [restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-12-18 02:34:54.357  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : Started App in 7.003 seconds (JVM running for 7.844)
2021-12-18 02:35:00.566  INFO 46077 --- [SpringApplicationShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2021-12-18 02:35:00.572  INFO 46077 --- [SpringApplicationShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2021-12-18 02:35:00.881  INFO 46077 --- [SpringApplicationShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
1
2
3
4
5
```

자동으로 다섯줄이 더 출력되는 것을 확인할 수 있습니다.

`-F` 옵션을 사용할 경우 spring.log 파일을 다른 파일명으로 변경하고 다시 작성해도 이어서 출력되는 것을 확인할 수 있습니다.

## wc

### Description

word count의 약자로서 파일에 word, line, character가 몇 개가 있는지 출력해주는 명령어 입니다.

### Options

자주 사용되는 옵션은 `-l` 으로 라인 수만 출력하는 옵션입니다.

설정파일 같은 파일의 라인의 수는 특정 정보를 나타낼 수 있으므로 자주 사용하는 옵션입니다.

### Examples

#### `wc spring.log`

```text
> wc spring.log
      43     541    6719 spring.log
```

43개의 라인, 541개의 단어, 6719개의 문자로 이루어져있음을 나타냅니다.

#### `wc -l spring.log`

```text
> wc -l spring.log
      43 spring.log
```

라인 수와 파일이름을 출력해줍니다.

라인 수만 얻고 싶으면 다른 명령어를 같이 사용해줘야 합니다.

```text
> wc -l spring.log | awk '{ print $1 }'
43
```

`awk` 명령어는 출력된 결과를 띄어쓰기 기준으로 파싱해서 토큰을 출력해주는 명령어 입니다. $1은 첫 번째 토큰을 의미하므로 라인 수만 출력되는 것을 확인할 수 있습니다.

유사한 명령어로 `cut`이 존재하는데 `awk`, `cut` 모두 이후 포스팅에서 다뤄보도록 하겠습니다.

#### `wc spring.log test.log`

```text
> wc spring.log test.log
      43     541    6719 spring.log
       4       5      29 test.log
      47     546    6748 total
```

여러 개의 파일을 넣으면 각각의 정보아 총 합계 정보가 같이 출력됩니다.

#### `wc *.log`

```text
> wc *.log
      43     541    6719 spring.log
       4       5      29 test.log
      47     546    6748 total
```

`*`를 이용해 같은 타입의 파일을 모두 확인할 수도 있습니다.

## nl

### Description

파일을 출력할 때 line number를 같이 출력해줍니다.

### Options

자주 사용되는 옵션은 아래와 같습니다.

* -ba: 모든 라인에 대해 넘버링
* -v N: 시작 라인 넘버를 지정
* -s: 라인 넘버 출력 후 출력할 separator 지정

### Examples

#### `nl spring.log`

```text
> nl spring.log
     1	2021-12-18 02:34:47.953  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : Starting App using Java 11.0.11 on Jaime-iMac.local with PID 46077 (/Users/jaime/git-repo/spring-boot-app/out/production/classes started by jaime in /Users/jaime/git-repo/spring-boot-app)
     2	2021-12-18 02:34:47.957  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : The following profiles are active: local
     3	2021-12-18 02:34:48.018  INFO 46077 --- [restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : Devtools property defaults active! Set 'spring.devtools.add-properties' to 'false' to disable
     4	2021-12-18 02:34:48.019  INFO 46077 --- [restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : For additional web related logging consider setting the 'logging.level.web' property to 'DEBUG'
     5	2021-12-18 02:34:49.148  INFO 46077 --- [restartedMain] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFAULT mode.
     6	2021-12-18 02:34:49.205  INFO 46077 --- [restartedMain] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 53 ms. Found 1 JPA repository interfaces.
     7	2021-12-18 02:34:49.850  INFO 46077 --- [restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
     8	2021-12-18 02:34:49.857  INFO 46077 --- [restartedMain] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
     9	2021-12-18 02:34:49.857  INFO 46077 --- [restartedMain] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.52]
    10	2021-12-18 02:34:49.918  INFO 46077 --- [restartedMain] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
    11	2021-12-18 02:34:49.918  INFO 46077 --- [restartedMain] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1898 ms
    12	2021-12-18 02:34:49.995  INFO 46077 --- [restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
    13	2021-12-18 02:34:52.348  INFO 46077 --- [restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
    14	2021-12-18 02:34:52.353  INFO 46077 --- [restartedMain] o.s.b.a.h2.H2ConsoleAutoConfiguration    : H2 console available at '/h2-console'. Database available at 'jdbc:h2:file:./testdb'
    15	2021-12-18 02:34:52.486  INFO 46077 --- [restartedMain] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
    16	2021-12-18 02:34:52.509  INFO 46077 --- [restartedMain] org.hibernate.Version                    : HHH000412: Hibernate ORM core version 5.4.32.Final
    17	2021-12-18 02:34:52.578  INFO 46077 --- [restartedMain] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.1.2.Final}
    18	2021-12-18 02:34:52.637  INFO 46077 --- [restartedMain] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
    19	2021-12-18 02:34:53.137  INFO 46077 --- [restartedMain] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
    20	2021-12-18 02:34:53.142  INFO 46077 --- [restartedMain] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
    21	2021-12-18 02:34:53.575  WARN 46077 --- [restartedMain] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
    22	2021-12-18 02:34:53.708  INFO 46077 --- [restartedMain] .s.s.UserDetailsServiceAutoConfiguration :

    23	Using generated security password: 9c32255b-378b-426f-b121-a707a723c2c2

    24	2021-12-18 02:34:53.781  INFO 46077 --- [restartedMain] o.s.b.a.w.s.WelcomePageHandlerMapping    : Adding welcome page template: index
    25	2021-12-18 02:34:53.947  INFO 46077 --- [restartedMain] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 1 endpoint(s) beneath base path '/actuator'
    26	2021-12-18 02:34:53.965  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure org.springframework.boot.autoconfigure.security.servlet.StaticResourceRequest$StaticResourceRequestMatcher@33f240e4 with []
    27	2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Mvc [pattern='/node_modules/**'] with []
    28	2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Mvc [pattern='/images/**'] with []
    29	2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Ant [pattern='/h2-console/**'] with []
    30	2021-12-18 02:34:53.991  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure any request with [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@6117e470, org.springframework.security.web.context.SecurityContextPersistenceFilter@12046c24, org.springframework.security.web.header.HeaderWriterFilter@11aa1ef1, org.springframework.security.web.csrf.CsrfFilter@2e02b351, org.springframework.security.web.authentication.logout.LogoutFilter@7a92e469, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@70debf5b, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@4b4e2362, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@427b3465, org.springframework.security.web.session.SessionManagementFilter@77e75116, org.springframework.security.web.access.ExceptionTranslationFilter@14c7b847, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@2406e5cf]
    31	2021-12-18 02:34:54.289  INFO 46077 --- [restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
    32	2021-12-18 02:34:54.330  INFO 46077 --- [restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
    33	2021-12-18 02:34:54.357  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : Started App in 7.003 seconds (JVM running for 7.844)
    34	2021-12-18 02:35:00.566  INFO 46077 --- [SpringApplicationShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
    35	2021-12-18 02:35:00.572  INFO 46077 --- [SpringApplicationShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
    36	2021-12-18 02:35:00.881  INFO 46077 --- [SpringApplicationShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
    37	1
    38	2
    39	3
    40	4
    41	5
```

앞에 숫자가 출력된 것을 확인할 수 있습니다.

22번과 23번, 23번과 24번 사이 공백이 있는데 여기는 넘버링이 안 되어있는 것을 확인할 수 있는데요, 아래 옵션을 사용하시면 공백을 포함해서 출력하게 됩니다.

#### `nl -ba spring.log`

```text
> ~/git-repo/lcalmsky/resources/file/ [feature/1+*] nl -ba spring.log
     1	2021-12-18 02:34:47.953  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : Starting App using Java 11.0.11 on Jaime-iMac.local with PID 46077 (/Users/jaime/git-repo/spring-boot-app/out/production/classes started by jaime in /Users/jaime/git-repo/spring-boot-app)
     2	2021-12-18 02:34:47.957  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : The following profiles are active: local
     3	2021-12-18 02:34:48.018  INFO 46077 --- [restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : Devtools property defaults active! Set 'spring.devtools.add-properties' to 'false' to disable
     4	2021-12-18 02:34:48.019  INFO 46077 --- [restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : For additional web related logging consider setting the 'logging.level.web' property to 'DEBUG'
     5	2021-12-18 02:34:49.148  INFO 46077 --- [restartedMain] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFAULT mode.
     6	2021-12-18 02:34:49.205  INFO 46077 --- [restartedMain] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 53 ms. Found 1 JPA repository interfaces.
     7	2021-12-18 02:34:49.850  INFO 46077 --- [restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
     8	2021-12-18 02:34:49.857  INFO 46077 --- [restartedMain] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
     9	2021-12-18 02:34:49.857  INFO 46077 --- [restartedMain] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.52]
    10	2021-12-18 02:34:49.918  INFO 46077 --- [restartedMain] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
    11	2021-12-18 02:34:49.918  INFO 46077 --- [restartedMain] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1898 ms
    12	2021-12-18 02:34:49.995  INFO 46077 --- [restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
    13	2021-12-18 02:34:52.348  INFO 46077 --- [restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
    14	2021-12-18 02:34:52.353  INFO 46077 --- [restartedMain] o.s.b.a.h2.H2ConsoleAutoConfiguration    : H2 console available at '/h2-console'. Database available at 'jdbc:h2:file:./testdb'
    15	2021-12-18 02:34:52.486  INFO 46077 --- [restartedMain] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
    16	2021-12-18 02:34:52.509  INFO 46077 --- [restartedMain] org.hibernate.Version                    : HHH000412: Hibernate ORM core version 5.4.32.Final
    17	2021-12-18 02:34:52.578  INFO 46077 --- [restartedMain] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.1.2.Final}
    18	2021-12-18 02:34:52.637  INFO 46077 --- [restartedMain] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
    19	2021-12-18 02:34:53.137  INFO 46077 --- [restartedMain] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
    20	2021-12-18 02:34:53.142  INFO 46077 --- [restartedMain] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
    21	2021-12-18 02:34:53.575  WARN 46077 --- [restartedMain] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
    22	2021-12-18 02:34:53.708  INFO 46077 --- [restartedMain] .s.s.UserDetailsServiceAutoConfiguration :
    23
    24	Using generated security password: 9c32255b-378b-426f-b121-a707a723c2c2
    25
    26	2021-12-18 02:34:53.781  INFO 46077 --- [restartedMain] o.s.b.a.w.s.WelcomePageHandlerMapping    : Adding welcome page template: index
    27	2021-12-18 02:34:53.947  INFO 46077 --- [restartedMain] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 1 endpoint(s) beneath base path '/actuator'
    28	2021-12-18 02:34:53.965  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure org.springframework.boot.autoconfigure.security.servlet.StaticResourceRequest$StaticResourceRequestMatcher@33f240e4 with []
    29	2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Mvc [pattern='/node_modules/**'] with []
    30	2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Mvc [pattern='/images/**'] with []
    31	2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Ant [pattern='/h2-console/**'] with []
    32	2021-12-18 02:34:53.991  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure any request with [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@6117e470, org.springframework.security.web.context.SecurityContextPersistenceFilter@12046c24, org.springframework.security.web.header.HeaderWriterFilter@11aa1ef1, org.springframework.security.web.csrf.CsrfFilter@2e02b351, org.springframework.security.web.authentication.logout.LogoutFilter@7a92e469, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@70debf5b, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@4b4e2362, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@427b3465, org.springframework.security.web.session.SessionManagementFilter@77e75116, org.springframework.security.web.access.ExceptionTranslationFilter@14c7b847, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@2406e5cf]
    33	2021-12-18 02:34:54.289  INFO 46077 --- [restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
    34	2021-12-18 02:34:54.330  INFO 46077 --- [restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
    35	2021-12-18 02:34:54.357  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : Started App in 7.003 seconds (JVM running for 7.844)
    36	2021-12-18 02:35:00.566  INFO 46077 --- [SpringApplicationShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
    37	2021-12-18 02:35:00.572  INFO 46077 --- [SpringApplicationShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
    38	2021-12-18 02:35:00.881  INFO 46077 --- [SpringApplicationShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
    39	1
    40	2
    41	3
    42	4
    43	5
```

#### `nl -s "] " spring.log`

```text
> nl -s "] " spring.log
     1] 2021-12-18 02:34:47.953  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : Starting App using Java 11.0.11 on Jaime-iMac.local with PID 46077 (/Users/jaime/git-repo/spring-boot-app/out/production/classes started by jaime in /Users/jaime/git-repo/spring-boot-app)
     2] 2021-12-18 02:34:47.957  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : The following profiles are active: local
     3] 2021-12-18 02:34:48.018  INFO 46077 --- [restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : Devtools property defaults active! Set 'spring.devtools.add-properties' to 'false' to disable
     4] 2021-12-18 02:34:48.019  INFO 46077 --- [restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : For additional web related logging consider setting the 'logging.level.web' property to 'DEBUG'
     5] 2021-12-18 02:34:49.148  INFO 46077 --- [restartedMain] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFAULT mode.
     6] 2021-12-18 02:34:49.205  INFO 46077 --- [restartedMain] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 53 ms. Found 1 JPA repository interfaces.
     7] 2021-12-18 02:34:49.850  INFO 46077 --- [restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
     8] 2021-12-18 02:34:49.857  INFO 46077 --- [restartedMain] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
     9] 2021-12-18 02:34:49.857  INFO 46077 --- [restartedMain] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.52]
    10] 2021-12-18 02:34:49.918  INFO 46077 --- [restartedMain] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
    11] 2021-12-18 02:34:49.918  INFO 46077 --- [restartedMain] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1898 ms
    12] 2021-12-18 02:34:49.995  INFO 46077 --- [restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
    13] 2021-12-18 02:34:52.348  INFO 46077 --- [restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
    14] 2021-12-18 02:34:52.353  INFO 46077 --- [restartedMain] o.s.b.a.h2.H2ConsoleAutoConfiguration    : H2 console available at '/h2-console'. Database available at 'jdbc:h2:file:./testdb'
    15] 2021-12-18 02:34:52.486  INFO 46077 --- [restartedMain] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
    16] 2021-12-18 02:34:52.509  INFO 46077 --- [restartedMain] org.hibernate.Version                    : HHH000412: Hibernate ORM core version 5.4.32.Final
    17] 2021-12-18 02:34:52.578  INFO 46077 --- [restartedMain] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.1.2.Final}
    18] 2021-12-18 02:34:52.637  INFO 46077 --- [restartedMain] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
    19] 2021-12-18 02:34:53.137  INFO 46077 --- [restartedMain] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
    20] 2021-12-18 02:34:53.142  INFO 46077 --- [restartedMain] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
    21] 2021-12-18 02:34:53.575  WARN 46077 --- [restartedMain] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
    22] 2021-12-18 02:34:53.708  INFO 46077 --- [restartedMain] .s.s.UserDetailsServiceAutoConfiguration :
      ]
    23] Using generated security password: 9c32255b-378b-426f-b121-a707a723c2c2
      ]
    24] 2021-12-18 02:34:53.781  INFO 46077 --- [restartedMain] o.s.b.a.w.s.WelcomePageHandlerMapping    : Adding welcome page template: index
    25] 2021-12-18 02:34:53.947  INFO 46077 --- [restartedMain] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 1 endpoint(s) beneath base path '/actuator'
    26] 2021-12-18 02:34:53.965  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure org.springframework.boot.autoconfigure.security.servlet.StaticResourceRequest$StaticResourceRequestMatcher@33f240e4 with []
    27] 2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Mvc [pattern='/node_modules/**'] with []
    28] 2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Mvc [pattern='/images/**'] with []
    29] 2021-12-18 02:34:53.966  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure Ant [pattern='/h2-console/**'] with []
    30] 2021-12-18 02:34:53.991  INFO 46077 --- [restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Will secure any request with [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@6117e470, org.springframework.security.web.context.SecurityContextPersistenceFilter@12046c24, org.springframework.security.web.header.HeaderWriterFilter@11aa1ef1, org.springframework.security.web.csrf.CsrfFilter@2e02b351, org.springframework.security.web.authentication.logout.LogoutFilter@7a92e469, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@70debf5b, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@4b4e2362, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@427b3465, org.springframework.security.web.session.SessionManagementFilter@77e75116, org.springframework.security.web.access.ExceptionTranslationFilter@14c7b847, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@2406e5cf]
    31] 2021-12-18 02:34:54.289  INFO 46077 --- [restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
    32] 2021-12-18 02:34:54.330  INFO 46077 --- [restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
    33] 2021-12-18 02:34:54.357  INFO 46077 --- [restartedMain] io.lcalmsky.app.App                      : Started App in 7.003 seconds (JVM running for 7.844)
    34] 2021-12-18 02:35:00.566  INFO 46077 --- [SpringApplicationShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
    35] 2021-12-18 02:35:00.572  INFO 46077 --- [SpringApplicationShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
    36] 2021-12-18 02:35:00.881  INFO 46077 --- [SpringApplicationShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
    37] 1
    38] 2
    39] 3
    40] 4
    41] 5
```

라인 넘버 뒤에 "] "가 붙은 것을 확인할 수 있습니다.

---

여기까지 `head`, `tail`, `wc`, `nl` 네 가지 명령어에 대해 알아봤습니다.

다음 포스팅에서는 보다 복잡하고 강력한 텍스트 처리 명렁에 대해 알아보겠습니다.