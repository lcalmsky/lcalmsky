스프링 프레임워크를 이용해 개발하면서 커스텀 애너테이션을 사용해야할 때가 있습니다.

저는 최근 헥사고날 아키텍처를 적용하면서 어댑터, 유스케이스 등 기존의 `@Controller`, `@Service`와 매핑될만한 클래스들에 대해 커스텀 애너테이션을 사용하고 있는데요, 사용할 때 기존 기능을 유지해야하기 때문에 보통 기존 애너테이션을 커스텀 애너테이션에 추가하여 사용합니다.

```java
package io.lcalmsky.demo.infrastructure.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.web.bind.annotation.RestController;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@RestController
public @interface WebAdapter {

}
```

이런식으로 작성한 뒤에 실제로 HTTP 요청을 처리할 컨트롤러에 해당하는 구현체에 애너테이션을 붙여서 사용할 수 있습니다.

```java
package io.lcalmsky.demo.modules.foo.user.adapter.in.web;

import io.lcalmsky.demo.infrastructure.annotation.WebAdapter;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.RequestMapping;

@WebAdapter
@RequestMapping(path = "/api/v1/foo/bar")
@RequiredArgsConstructor
public class FooController {

  // 생략
}
```

이렇게 해도 어댑터이구나 하는 정도는 누구나 알 수 있는 인터페이스 역할은 충분히 할 수 있는데요, 만약에 컴포넌트 이름(qualifier)을 직접 지정하려면 `@Qualifier`와 같은 애너테이션을 추가해줘야 합니다.

그리고 `@RequestMapping` 애너테이션도 추가되어있어 다소 지저분해 보일 수 있습니다.

이런 문제를 한번에 해결해주는 게 바로 `@AliasFor` 애너테이션 입니다.

`@AliasFor` 애너테이션의 설명부터 보자면,

> @AliasFor is an annotation that is used to declare aliases for annotation attributes.

애너테이션의 attribute를 위한 별칭을 선언하기 위해 사용합니다.

글로만 보면 헷갈리는데 코드를 보면서 다시 확인해보겠습니다.

```java
package io.lcalmsky.demo.infrastructure.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.core.annotation.AliasFor;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@RestController
@RequestMapping
public @interface WebAdapter {

  @AliasFor(annotation = RestController.class) // (1)
  String value() default "";

  @AliasFor(annotation = RequestMapping.class, attribute = "path") // (2)
  String path() default "";

}
```

1번은 `@RestController`의 동일 attribute인 `value`를 위한 별칭을 선언한 것입니다.

`@WebAdapter("foo")`라고 코드에 명시했을 때 `value = "foo"`가 되고 `@RestController`의 `value`에 전달됩니다.

> value는 다른 attribute가 없을 때 자신의 attribute를 명시하지 않아도 되기 때문에 바로 값을 할당하였습니다.

`@RestController`의 `value`를 추적해보면,

```java
package org.springframework.web.bind.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.core.annotation.AliasFor;
import org.springframework.stereotype.Controller;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController {

 @AliasFor(annotation = Controller.class)
 String value() default "";

}
```

이렇게 다시 `@Controller`의 `value`의 별칭으로 사용하고 있습니다.

마찬가지로 `@Controller`의 `value`를 보면 `@Component`의 `value`의 별칭으로 사용하고 있음을 확인할 수 있는데요, `@Component`에서는 `value`를 `name`으로, 즉 `qualifer`로 사용합니다.

앞서 `@Qualifier`를 지정해야했던 수고를 덜 수 있습니다.

2번의 경우 `@RequestMapping` 애너테이션의 `path` attribute의 별칭을 지정한 것입니다.

`@RequestMapping`의 `path`는 잘 아시다시피 요청 `uri`를 명시할 수 있습니다.

굳이 추가로 애너테이션을 달지 않아도 하나의 커스텀 애너테이션에 두 가지 속성을 지정할 수 있게 되었습니다.

`@WebAdapter`를 사용하는 쪽에서는 아래 처럼 사용할 수 있습니다.

```java
@WebAdapter(value = "fooAdapter", path = "/api/v1/foo/bar")
```

---

`@AliasFor` 애너테이션을 활용하면 깔끔하게 그 역할에 맞는 attribute들을 커스텀 애너테이션에 추가할 수 있습니다.