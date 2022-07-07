## Overview

JPA를 사용하느라 `Entity` 클래스를 정의하다보면 `static` 생성자를 만들어야 할 때가 있습니다.

특히 외부에서 객체를 생성할 수 없게 `protected` 레벨로 기본 생성자를 만들고, `Dirty Checking` 방지를 위해 `setter` 또한 외부로 공개하지 않습니다.

`lombok`을 같이 사용하면서 필요한 생성자를 만들고 `@Builder`를 생성자에 추가해 외부에서 객체를 생성할 수 있게 하는 경우도 있지만 `Builder` 패턴을 이용하면 필수항목과 선택적인 항목을 구분하기 어렵고, 필수항목을 따로 받을 수 있는 방법이 있긴한데 깔끔하게 잘 되진 않습니다.

이런 상황에서 다들 어떻게 개발하시나요?

`IntelliJ`에 생성자를 기반으로 `static` 생성자를 만들어주는 기능이 있습니다. 

> Builder도 만들어주지만 롬복보다 불편하기 때문에 pass!

## static 생성자 만들기

먼저 `Entity`를 하나 만들어보겠습니다.

```java
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.ToString;

@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@ToString
public class Member {

  @Id
  @GeneratedValue
  @Column(name = "member_id")
  private Long id;
  private String name;
  private String email;
  private Integer age;
  private String phoneNumber;

}
```

`id`를 제외하고 4개의 필드를 가진 `entity`를 생성하였습니다.

현재 외부에서 `Member` 객체를 만들 수 있는 방법이 없으므로 생성자가 필요합니다.

이름과 이메일이 필수항목이고 나이, 휴대폰 번호는 이후에 업데이트하는 기능을 제공한다고 할 때, 생성자를 아래 처럼 생성할 수 있습니다.

```java
  protected Member(String name, String email) {
    this.name = name;
    this.email = email;
  }
```

외부에서 `new`를 통해 객체 생명 주기에 관여할 수 없게 `protected` 레벨로 생성자를 만들었습니다.

이후 `static` 생성자를 만들어보면,

```java
  public static Member withNameAndEmail(String name, String email) {
    return new Member(name, email);
  }
```

이런식으로 `static` 생성자를 만들어 외부에 공개할 수 있고 외부에서는

```java
Member member = Member.withNameAndEmail("jaime", "lcalmsky@gmail.com");
```

이렇게 `Entity`를 생성할 수 있게 됩니다.

이후에 다른 필드가 추가되거나 다른 생성자가 추가되어야 하는 경우 이 작업을 계속 반복해줘야 하는데요, 여기서 `IntelliJ`가 제공하는 기능을 사용할 수 있습니다.

---

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@ToString
public class Member {

  @Id
  @GeneratedValue
  @Column(name = "member_id")
  private Long id;
  private String name;
  private String email;
  private Integer age;
  private String phoneNumber;

}
```

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/master/resources/image/docs-blog-intellij-003-01.png)

먼저 필드만 추가한 상태에서 `⌘ N` 을 눌러 `constructor`를 입력합니다. (윈도우 기준 `Alt+Insert`)

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/master/resources/image/docs-blog-intellij-003-02.png)

필요한 필드만 선택한 뒤 `OK`를 눌러 생성자가 생성된 것을 확인합니다.

```java
  public Member(String name, String email) {
    this.name = name;
    this.email = email;
  }
```

`name`과 `email`만 선택하고 `OK`를 눌렀을 때 위와 같은 `public` 생성자가 생성되었습니다.

생성자 위치로 커서를 옮긴 뒤에 `⌃ T`를 눌러 `Refactor This` 컨텍스트를 띄웁니다. (윈도우 기준 `Ctrl+Alt+Shift+T`)

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/master/resources/image/docs-blog-intellij-003-03.png)

메뉴 하단에 보면 `Replace Constructor with Factory Method...` 라는 메뉴가 있고 단축키는 `0`번으로 나와있습니다.

> IntelliJ 버전(라이선스)에 따라 `⌥ ⏎` 를 눌러야 해당 메뉴가 나타날 수도 있습니다.  
> (윈도우 기준 `Alt + Enter`)

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/master/resources/image/docs-blog-intellij-003-04.png)

해당 메뉴에 진입하면, 메서드 이름만 입력하면 되고 `static` 생성자를 생성해줌과 동시에 기존 생성자를 `private`으로 바꿔줍니다!

최종적으로 완성된 코드는 아래와 같습니다.

```java
package com.tistory.jaimenote.jpa.inheritance.entity;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.ToString;

@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@ToString
public class Member {

  @Id
  @GeneratedValue
  @Column(name = "member_id")
  private Long id;
  private String name;
  private String email;
  private Integer age;
  private String phoneNumber;

  private Member(String name, String email) {
    this.name = name;
    this.email = email;
  }

  public static Member withNameAndEmail(String name, String email) {
    return new Member(name, email);
  }
}
```

---

어때요, 참 쉽죠? 💁‍

`IntelliJ`는 쓰면 쓸수록 정말 좋은 `IDE`인 거 같습니다.