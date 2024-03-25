Heap pollution은 Java에서 제네릭스를 사용할 때 발생할 수 있는 문제 중 하나입니다.

제네릭스는 타입 안정성을 보장하기 위해 사용되는 기술이지만, 제네릭 타입이 적용된 컬렉션에 잘못된 타입의 객체를 삽입하거나, 타입이 일치하지 않는 객체를 반환하도록 코드가 작성된 경우 발생할 수 있습니다. 이로 인해 컴파일러에서는 오류가 발생하지 않지만, 런타임에서 예기치 않은 동작을 할 수 있습니다.

예를 들어, 다음과 같은 코드가 있다고 가정해 봅시다.

```java
List<String> stringList = new ArrayList<>();
List rawList = stringList; // warning: unchecked conversion
rawList.add(1); // heap pollution
String s = stringList.get(0); // ClassCastException
```
위 코드에서는 `List<String>` 타입의 `stringList` 객체를 생성하고, 이를 `List` 타입의 `rawList` 변수에 할당합니다. 이 때, rawList 변수는 제네릭 타입이 아닌 "Raw Type"으로 처리되므로, 컴파일러에서 "unchecked conversion" 경고가 발생합니다.

이후, `rawList`에 `Integer` 타입의 1을 추가하면, `stringList`에는 `String` 타입의 객체만 저장하도록 선언했음에도 불구하고, Integer 타입의 객체가 추가됩니다. 이러한 상황을 "heap pollution"이라고 부릅니다.

마지막으로, `stringList`에서 첫 번째 객체를 가져오려고 하면, `ClassCastException`이 발생합니다. 이는 `rawList`에 잘못된 타입의 객체가 추가됨으로써, `stringList`의 타입 안정성이 깨졌기 때문입니다.

따라서, 제네릭스를 사용할 때에는 이러한 "heap pollution" 문제에 주의해야 하며, 원시 타입(raw type)을 사용하는 것은 지양해야 합니다.