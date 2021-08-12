`JPA`를 사용할 때 `Entity`의 필드가 `Enum`인 경우, 실제 코딩시에는 `Enum`의 `Constant`를 사용하지만 `DB`에 저장할 땐 문자열 또는 정수로 된 코드를 저장하는 경우가 있을 수 있습니다.

이럴 때 커스텀 `Converter`를 만들어 `AttributeConverter`를 구현하여 해결하곤합니다.

그리고 `JSON`을 객체로 매핑할 때 해당 객체에 `Enum`이 포함된 경우에도 마찬가지로 비슷한 작업을 해줘야 합니다.

이 때 자주 사용되는 `static` 필드와 메서드가 있는데요, 예시를 하나 들어보자면,

```java
package io.lcalmsky.blogexamples.domain.entity.support;

import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonValue;
import lombok.Getter;

import javax.persistence.AttributeConverter;
import javax.persistence.Converter;
import java.util.Map;
import java.util.Optional;
import java.util.function.Function;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public enum Gender {
    MALE("M"),
    FEMALE("F");

    private static final Map<String, Gender> CODE_MAP = Stream.of(values())
            .collect(Collectors.toMap(Gender::getValue, Function.identity())); // (1)

    @JsonCreator // (2)
    public static Gender resolve(String value) {
        return Optional.ofNullable(CODE_MAP.get(value))
                .orElseThrow(() -> new IllegalArgumentException("invalid value"));
    }

    @Converter // (3)
    public static final class GenderConverter implements AttributeConverter<Gender, String> {
        @Override
        public String convertToDatabaseColumn(Gender attribute) { // (4)
            return attribute.getValue();
        }

        @Override
        public Gender convertToEntityAttribute(String dbData) { // (5)
            return Gender.resolve(dbData);
        }
    }

    @Getter 
    @JsonValue // (6)
    private final String value;

    Gender(String value) {
        this.value = value;
    }
}
```

이렇게 `Gender` 라는 `enum`을 정의했다고 가정하고 주석 처리 된 번호에 대해 각각 설명하겠습니다.

> (1) "M", "F"를 키로, `Gender`를 값으로 가지는 맵을 만듧니다.  
> (2) `@JsonCreator` 애너테이션이 있으면 `JSON`을 `deserialize` 할 때 해당 메서드를 이용해 값을 변환합니다. 예를 들어 `{"gender": "M"}` 이라는 `JSON`을 변환하여 소스 코드 내에서는 `Gender.MALE`로 사용할 수 있게 해줍니다.  
> (3) `@Converter` 애너테이션이 있으면 `JPA`에서 `DB`와 연동할 때 해당 `Converter`를 이용해 값을 변환해 줍니다.  
> (4) `enum`을 `DB`에 저장할 값으로 변환합니다.  
> (5) `DB`에 있는 값을 다시 `enum`으로 변환합니다.  
> (6) `JSON`으로 `serialize` 할 때 `enum` 값을 해당 값으로 변환합니다.

예시로 든 `enum`이 하필 `Gender`라 항목이 두 개 밖에 없습니다.

하지만 개발을 하다보면 코드로 정의하여 주고 받는 경우가 매우 많고, 개발할 때 enum 등오로 정의해놓지 않으면 매우매우 불편한 상황들이 자주 연출되곤 합니다.

예를 들어 `01`이란 코드가 `정상`, `02`란 코드가 `심사 필요`, `03`은 `차단`, `04`는 `다른 프로세스 수행`, `05`는 `심사 완료` 등 숫자만 보고는 알 수 없는 내용들을 매핑하여 규격에 포함시키는 경우가 비일비재하게 일어나는데요, 이런 경우 `enum`으로 매핑하면 헷갈리지 않고 보다 가독성있게 개발할 수 있고 유지보수도 용이하게 할 수 있습니다.

또한 객체지향적으로 `abstract` 메서드나 `interface`를 구현하도록하여 해당 `enum`의 `constant` 별로 같은 메서드를 호출해 각각의 성격에 맞는 작업을 하도록 설계하여 매 번 조건문을 추가하지 않게도 개발할 수 있습니다.

따라서 허접한 예시와는 별개로 `enum`을 정의하여 사용하는 것은 매우 중요하고, 매 번 사용하는 보일러 플레이트 코드들을 간단히 작성할 수 있으면 더더욱 좋겠죠.

위의 소스 코드를 예시로 들면, (1), (2), (3) 번이 보일러 플레이트 코드라고 볼 수 있습니다. (6)의 경우 필드의 타입이 `String`이 아닐 수도 있고 변수 이름이 `value`가 아닐 수 있어 공통 코드에서는 제외하였습니다.

[이전 포스팅](https://jaime-note.tistory.com/47?category=849443)에서 `Entity`를 자동화시킨 것 처럼 보일러 플레이트 코드들을 간단히 작성할 수 있게 해보려고 합니다.

---

> 저는 macOS와 IntelliJ IDEA 2021.2 Ultimate Edition을 사용하고 있습니다.    
> 저와 동일한 환경에서 작업하시는 게 아닐 경우 UI 등이 상이할 수 있습니다.

먼저 `CMD` + `,` 를 눌러 `Preferences`로 진입합니다.

`Live Templates`를 검색하여 해당 메뉴로 들어가 `Java` 항목에서 `CMD` + `N`을 눌러 `Live Template`을 추가합니다.

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-002-01.png)

`Abbreviation`에 실제 사용할 약자를 넣고(저는 그냥 `enumsupport`라고 길게 적었습니다. 어차피 이것도 자동완성이 돼서..) `Description`에 설명을 적어줍니다.

그리고 `Template text`가 가장 중요한데, 이번에 작성하면서 알아낸 것이 `IntelliJ IDE`의 경우 `package`를 포함하여 클래스를 작성하게되면 자동으로 `import` 해준다고 합니다. 👏

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-002-02.png)

```java
private static final java.util.Map<$KEY$, $CLASS$> $FIELD$MAP = java.util.stream.Stream.of(values())
        .collect(java.util.stream.Collectors.toMap($CLASS$::$GETTER$, java.util.function.Function.identity()));

public static $CLASS$ resolve$FROM$($KEY$ $PARAM$) {
    return java.util.Optional.ofNullable($FIELD$MAP.get($PARAM$))
            .orElseThrow(() -> new IllegalArgumentException("$DESC$"));
}
```

`Template text`에서 `$` 사이에 작성한 것들은 변수로 취급되어 가공하거나 기본 값을 줄 수 있습니다.

이렇게 적고 `Edit variables`를 클릭하여 아래 캡쳐처럼 `CLASS`에만 `Expression`을 추가해줍니다. 

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-002-03.png)

각각의 변수에 대해 설명하자면,  
&nbsp;&nbsp;`KEY`는 맵에서 사용될 키 타입으로 JSON이나 DB에서 실제로 사용할 값의 타입 입니다.  
&nbsp;&nbsp;`CLASS`는 현재 `enum`의 이름(`className()`사용)이고,  
&nbsp;&nbsp;`FIELD`는 상수 `MAP` 앞에 사용할 `Prefix`인데 주로 필드와 매핑될 거 같아서 변수 이름을 `FIELD`로 설정하였습니다.  
&nbsp;&nbsp;`GETTER`는 메서드 레퍼런스에 추가할 실제 코드 값을 반환하는 `getter` 메서드를 넣는 변수,  
&nbsp;&nbsp;`FROM`은 `resolveFromFoo` 이런식으로 `resolve` 메서드가 많아질 것을 대비해 만든 변수입니다.  
&nbsp;&nbsp;`PARAM`은 `resolve` 호출할 때 파라미터 변수명,   
&nbsp;&nbsp;`DESC`는 에러 발생시 설명을 위한 변수입니다.

그리고나서 캡쳐의 좌측 하단에 `Define` 버튼을 눌러 `Java` - `Declaration`을 선택해줍니다.

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-002-04.png)

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-002-05.png)

이제 `static` 필드와 메서드 생성은 완료했고 한 김에 `Converter`도 간편하게 만들 수 있는 템플릿을 작성해보겠습니다.

일부러 변수명을 위에서 만든 것과 동일하게 하였기 때문에 간단히 캡쳐로 설명을 대체하겠습니다.

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-002-06.png)

![](https://raw.githubusercontent.com/lcalmsky/lcalmsky/main/resources/image/docs-blog-intellij-002-07.png)

```java
@javax.persistence.Converter
public static final class $CLASS$Converter implements javax.persistence.AttributeConverter<$CLASS$, $KEY$> {
    @Override
    public $KEY$ convertToDatabaseColumn($CLASS$ attribute) {
        return attribute.$GETTER$;
    }

    @Override
    public $CLASS$ convertToEntityAttribute($KEY$ dbData) {
        return $CLASS$.resolve$FROM$(dbData);
    }
}
```

이제 `Live Template` 추가를 모두 마쳤는데요, 마지막으로 시연하는 모습을 보여드리면서 포스팅을 마치겠습니다. 👋

(참고로 변수간 이동은 `TAB`을 누르면 되고 마지막 변수에서 `TAB`을 누르면 템플릿 작성 모드를 빠져나갑니다. 첨부한 영상에서는 `@JsonCreator` 등을 따로 추가하진 않았습니다.)