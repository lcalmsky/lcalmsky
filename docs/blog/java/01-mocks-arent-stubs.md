> 본 포스팅은 마틴 파울러의 [원글](https://martinfowler.com/articles/mocksArentStubs.html)을 번역한 것입니다.  
> 의역 및 오역이 있을 수 있으므로 원글로도 읽어보시는 것을 권장합니다.

`Mock`이라는 용어는 테스트를 위해 실제 객체를 모방한 특수한 객체를 설명하는 데 널리 사용되었습니다. 현재 대부분의 언어 환경에는 `Mock` 객체를 쉽게 생성하기 위한 프레임워크가 있습니다. 그러나 `Mock` 객체는 특수한 경우의 테스트를 가능하게하는 하나의 형태에 불과하다는 것을 우리는 종종 깨닫지 못하곤 합니다. 이 글에서는 Mock 객체가 어떻게 작동하는지, 어떻게 동작 검증 기반 테스트를 장려하는지, 관련된 커뮤니티에서 다른 스타일의 테스트를 개발하기 위해 `Mock` 객체를 어떻게 사용하는지를 설명합니다.  

`Mock`이라는 용어가 처음 등장한 이후로 현재는 매우 쉽게 접할 수 있는 용어가 되었지만, 종종 제대로 설명되지 않은 경우가 있습니다. 특히 테스트 환경의 공통적인 도우미 역할을 하는 `Stub`과 혼동되는 경우가 많습니다. 이 두 용어의 차이는 두 가지로 구분할 수 있습니다. 테스트 결과가 검증되는 방식(상태 검증과 행동 검증)과 테스트와 설계가 함께 동작하는 방식의 차이라고 볼 수 있고 이것을 테스트 주도 개발의 고전적이고 모조주의적 스타일이라고 부릅니다.

## 일반적인 테스트

간단한 예를 들어 두 가지 스타일을 설명하겠습니다. `Order` 객체를 주문하여 `Warehouse` 객체를 채우려고 합니다. `Order`는 하나의 제품과 수량으로 구성된 매우 간단한 객체입니다. `Warehouse`는 다양한 제품의 재고가 있습니다. `Warehouse`를 채우기위해 주문을 요청할 때 두 가지 가능한 응답이 있습니다. `Order`를 처리할 만큼의 제품이 `Warehouse`에 있으면 주문이 완료되고 `Warehouse`의 상품 수량은 해당 수량만큼 감소합니다. `Warehouse`에 제품이 충분하지 않으면 `Order`가 수행되지 않고 창고에는 아무 일도 일어나지 않습니다.

이 두 가지 동작은 몇 가지 테스트를 의미하며 이는 꽤 일반적인 JUnit 테스트처럼 보입니다.

```java
public class OrderStateTester extends TestCase {
    private static String TALISKER = "Talisker";
    private static String HIGHLAND_PARK = "Highland Park";
    private Warehouse warehouse = new WarehouseImpl();

    protected void setUp() throws Exception {
        warehouse.add(TALISKER, 50);
        warehouse.add(HIGHLAND_PARK, 25);
    }

    public void testOrderIsFilledIfEnoughInWarehouse() {
        Order order = new Order(TALISKER, 50);
        order.fill(warehouse);
        assertTrue(order.isFilled());
        assertEquals(0, warehouse.getInventory(TALISKER));
    }

    public void testOrderDoesNotRemoveIfNotEnough() {
        Order order = new Order(TALISKER, 51);
        order.fill(warehouse);
        assertFalse(order.isFilled());
        assertEquals(50, warehouse.getInventory(TALISKER));
    }
}
```

`xUnit` 테스트는 설정(setup), 실행(exercise), 검증(verify), 분해(teardown)의 일반적인 4단계 순서를 따릅니다. 이 경우 설정 단계는 부분적으로는 `setUp` 메서드에서(warehouse 설정), 부분적으로는 테스트 메서드에서(order 설정) 수행됩니다. 실행 단계에서는 `order.fill` 메서드가 호출됩니다. 이것은 우리가 테스트하고 싶은 일들을 할 수 있게 객체에게 수행을 명령하는 구간입니다. assert 문은 검증 단계이고 실행된 메서드가 작업을 올바르게 수행했는지 확인합니다. 이 테스트의 경우 분해 단계가 존재하지 않고 가비지 컬렉터가 암시적으로 이를 수행합니다.

설정 단계에서 두 개의 객체가 함께 사용됩니다. `Order`는 테스트중인 클래스이지만 `Order.fill`이 작동하려면 Warehouse 객체도 필요합니다. 이 상황에서 Order는 테스트에 중점을 둔 객체입니다. 테스트 지향적인 사람들은 테스트 대상 객체 또는 테스트 대상 시스템과 같은 용어를 사용하여 이러한 이름을 지정하는 것을 좋아합니다. Meszaros를 따라 `SUT` (System Under Test)라는 용어를 사용하겠습니다.

이 테스트를 위해서는 `SUT`(Order)와 `Collaborator`(Warehouse)가 필요합니다. `Warehouse`가 필요한 두 가지 이유는 다음과 같습니다. 먼저 테스트 된 동작이 동작하게 하기 위함이고, 다음으로는 검증하기 위함입니다. 이 주제를 더 자세히 살펴보면 `SUT`와 `Collaborator`를 구분할 수 있음을 알 수 있습니다.

이런 테스트 스타일은 상태 검증을 위해 사용합니다. 즉, 메서드가 실행된 후 `SUT`와 `Collaborator`의 상태를 검사하여 실행된 메서드가 올바르게 작동했는지 여부를 결정합니다. `Mock` 객체는 검증에 대한 다른 접근 방식을 가능하게 합니다.

## Mock 객체를 이용한 테스트

이제 동일한 동작을 `Mock` 객체를 이용해 테스트해보겠습니다. 이 코드에서는 `Mock`을 정의하기 위해 `jMock`이라는 라이브러리를 사용합니다.

```java
public class OrderInteractionTester extends MockObjectTestCase {
    private static String TALISKER = "Talisker";

    public void testFillingRemovesInventoryIfInStock() {
        //setup - data
        Order order = new Order(TALISKER, 50);
        Mock warehouseMock = new Mock(Warehouse.class);

        //setup - expectations
        warehouseMock.expects(once()).method("hasInventory")
                .with(eq(TALISKER), eq(50))
                .will(returnValue(true));
        warehouseMock.expects(once()).method("remove")
                .with(eq(TALISKER), eq(50))
                .after("hasInventory");

        //exercise
        order.fill((Warehouse) warehouseMock.proxy());

        //verify
        warehouseMock.verify();
        assertTrue(order.isFilled());
    }

    public void testFillingDoesNotRemoveIfNotEnoughInStock() {
        Order order = new Order(TALISKER, 51);
        Mock warehouse = mock(Warehouse.class);

        warehouse.expects(once()).method("hasInventory")
                .withAnyArguments()
                .will(returnValue(false));

        order.fill((Warehouse) warehouse.proxy());

        assertFalse(order.isFilled());
    }
}
```

`testFillingRemovesInventoryIfInStock`에 먼저 집중해봅시다.

우선 설정 단계가 매우 다릅니다. 처음은 데이터와 기대치의 두 부분으로 나뉩니다. 데이터 부분은 데이터를 가지고 작업할 객체를 설정합니다. 그런 의미에서 기존의 설정 부분과 유사합니다. 차이점은 생성되는 객체에 있습니다. `SUT`는 동일하지만 `Collaborator`는 그냥 `Warehouse` 객체가 아닌 `Mock Warehouse` 객체-기술적으로 `Mock` 클래스의 인스턴스-입니다.

설정의 두 번째 부분은 `Mock` 객체에 대한 기대치를 생성합니다. 기대치는 `SUT`가 실행될 때 `Mock` 객체에서 호출되어야하는 메서드를 나타냅니다.

모든 기대치가 설정되었으면 `SUT`를 실행합니다. 실행 후에는 두 가지 측면으로 검증합니다. `SUT`는 이전 테스트와 마찬가지로 검증하지만 `Mock` 객체 또한 기대에 맞게 호출되었는지 확인합니다. 

여기서 주요 차이점은 `Order`가 `Warehouse`와 상호작용할 때 올바른 작업을 수행했는지 확인하는 방법입니다. 상태 확인을 사용하면 `Warehouse`의 상태에 대해 검증합니다. `Mock`은 `Order`가 `Warehouse`에서 올바른 호출을 했는지 확인하는 행동 검증을 사용합니다. 우리는 설정 단계에서 `Mock`에게 우리가 무엇을 기대하고있는지 알려주고, 검증단계에서 `Mock`이 스스로 그것을 검증하게 합니다. 오직 `Order`만 `assert`를 사용해 확인하는데 메서드가 `Order`의 상태를 변경하지 않는다면 `Order`를 `assert`할 필요가 없습니다.

두 번째 테스트에서는 몇 가지 다른 작업을 수행합니다. 먼저 생성자가 아닌 `MockObjectTestCase`의 모의 메서드를 사용하여 `Mock`을 다르게 만듭니다. 이 방법은 나중에 명시적으로 `verify`를 호출할 필요가 없고, 생성된 `mock`은 테스트가 끝날 때 자동으로 확인됩니다. 첫 번째 테스트에서도 이 방법을 수행할 수 있었지만 모의 테스트가 작동하는 방식을 보여주기 위해 검증을 보다 명시적으로 보여주기 위해 분리하였습니다.

또 다른 점은 `withAnyArguments`를 사용하여 기대에 대한 제약을 완화하였다는 것입니다. 그 이유는 첫 번째 테스트에서 번호가 `Warehouse`로 전달되었는지 확인하므로 두 번째 테스트에서는 해당 사항을 반복할 필요가 없기 때문입니다. 나중에 순서의 논리를 변경해야 하는 경우 하나의 테스트만 실패하므로 테스트를 마이그레이션하는 수고를 덜 수 있습니다.

## Mock과 Stub의 차이

처음 도입되었을 때 많은 사람들이 `Stub`을 사용하는 일반적인 테스트 개념과 `Mock` 객체를 쉽게 혼동했습니다. 현재는 예전보다는 차이점을 이해하고 있는 사람들이 많아졌습니다. 그러나 `Mock`을 사용하는 방식을 완전히 이해하려면 `Mock`과 다른 종류의 `Test Double`을 이해하는 것이 중요합니다.

> Test Double은 xUnit Test Patterns의 저자인 제라드 메스자로스가 만든 용어로 테스트를 진행하기 어려운 경우 이를 대신해 테스트를 진행할 수 있도록 만들어주는 객체를 말합니다.  
> 단어 자체의 뜻은 '대역'에 가깝고 스턴트맨과 유사한 의미를 가집니다.

테스트를 수행할 때 한 번에 소프트웨어의 한 요소에 초점을 맞추게 되므로 일반적인 용어는 `단위 테스트(Unit Test)`입니다. 문제는 하나의 유닛을 테스트하기 위해서는 종종 다른 유닛들이 필요하다는 것-이 예시에서는 `Warehouse`-입니다.

위에서 보여드린 두 가지 테스트 스타일에서 첫 번째 경우는 실제 `Warehouse` 객체를 사용하고 두 번째 경우는 `Mock Warehouse` 객체를 사용합니다. `Mock` 객체를 사용하지 않고도 실제 `Warehouse` 객체를 대신하여 이와 같이 테스트에 사용되는 다른 형태의 객체가 있습니다.

이것을 정의하는 단어는 `Stub`, `Mock`, `Fake`, `Dummy` 처럼 매우 여러 가지가 있습니다. 이 문서에서는 `Gerard Meszaros` 책의 어휘를 따를 것입니다.  

Meszaros는 테스트 목적으로 실제 객체 대신 사용되는 모든 종류의 가상 객체에 대한 일반 용어로 `Test Double`을 사용합니다. Meszaros는 5가지 종류의 `Test Double`을 정의하였습니다.

* `Dummy` 객체는 전달되지만 실제로 사용되지는 않습니다. 일반적으로 매개변수 목록을 채우는 데만 사용됩니다.
* `Fake` 객체는 실제로 작동하는 구현을 가지고 있지만 일반적으로 프로덕션에는 적합하지 않은 방식을 취합니다. (메모리 기반 데이터베이스가 좋은 예시)
* `Stub`은 테스트 중에 만들어진 호출에 미리 준비된 답변을 제공하고 일반적으로 테스트를 위해 프로그래밍된 것 외에는 전혀 응답하지 않습니다.
* `Spy`는 그들이 어떻게 호출되었는지에 따라 일부 정보를 기록하는 `Stub` 입니다.
* `Mock`은 호출될 것으로 예상되는 사양을 형성하는 기대값으로 미리 프로그래밍 된 객체입니다.

이 중에서 `Mock`만 행동 검증을 수행합니다. 다른 것들은 상태 검증에 사용될 수 있습니다. `Mock`은 실제로 `SUT`가 `Collaborator`와 커뮤니케이션한다고 믿도록 해야하기 때문에 실행단계에서 다른 `Test Double`처럼 사용되지만 설정 및 검증단계에서 차이가 있습니다.

`Test Double`을 더 알아보기 위해 예제를 확장하였습니다. 많은 사람들은 실제 객체가 작업하기 불편한 경우에만 `Test Double`을 사용합니다. `Test Double`을 위한 더 일반적인 테스트 케이스로 Order를 채우지 못한 경우 이메일 메시지를 보내는 것을 예로 들 수 있습니다. 문제는 테스트 중에 고객에게 실제 이메일 메시지를 보내고 싶지 않다는 것입니다. 그래서 우리는 우리가 직접 제어하고 조작할 수 있는 이메일 시스템의 `Test Double`을 만들어야 합니다. 

여기서 우리는 `Mock과` `Stub`의 차이점을 확인할 수 있습니다. 메일 동작에 대한 테스트를 작성하는 중이라면 아래와 같은 간단한 `Stub`을 작성할 수 있습니다.

```java
public interface MailService {

  public void send(Message msg);
}

public class MailServiceStub implements MailService {

  private List<Message> messages = new ArrayList<Message>();

  public void send(Message msg) {
    messages.add(msg);
  }

  public int numberSent() {
    return messages.size();
  }
}        
```

그리고 우리는 `Stub`을 이용해 다음과 같이 상태 검증을 할 수 있습니다.

```java
class OrderStateTester {

  public void testOrderSendsMailIfUnfilled() {
    Order order = new Order(TALISKER, 51);
    MailServiceStub mailer = new MailServiceStub();
    order.setMailer(mailer);
    order.fill(warehouse);
    assertEquals(1, mailer.numberSent());
  }
}
```

물론 이 테스트는 메시지가 전송되었다는 매우 간단한 테스트입니다. 올바른 사람에게 올바른 내용으로 전송되었는지 테스트하지 않았지만 `Stub` 사용의 요점을 설명하기 위한 것입니다.

`Mock`을 사용하면 이 테스트가 상당히 달라집니다.

```java
class OrderInteractionTester {

  public void testOrderSendsMailIfUnfilled() {
    Order order = new Order(TALISKER, 51);
    Mock warehouse = mock(Warehouse.class);
    Mock mailer = mock(MailService.class);
    order.setMailer((MailService) mailer.proxy());

    mailer.expects(once())
        .method("send");
    warehouse.expects(once())
        .method("hasInventory")
        .withAnyArguments()
        .will(returnValue(false));

    order.fill((Warehouse) warehouse.proxy());
  }
}
```

두 가지 경우 모두 실제 메일 서비스 대신 `Test Double`을 사용하고 있습니다. `Stub`은 상태 검증을, `Mock`은 행동 검증을 사용한다는 차이점이 있습니다.

`Stub`에서 상태 확인을 사용하려면 확인을 돕기 위해 `Stub`에서 몇 가지 추가 메서드를 만들어야 합니다. 결과적으로 `Stub`은 `MailService`를 구현하지만 추가적인 테스트 메서드를 구현해야합니다.

`Mock` 객체는 항상 동작 검증을 사용하며 `Stub`은 어떤 방식으로도 사용할 수 있습니다. Meszaros는 `Stub`을 행동 검증을 위해 사용되는 `Test Spy`라고 언급합니다. 차이점은 얼마나 정확하게 `Test Double`이 실행되고 검증되는지에 있습니다.

## 전통 방식과 Mock 방식 테스트

이제 두 번째 이분법인 전통과 모의 테스트 기반 개발에 대해 탐구할 수 있는 시점에 와있습니다. 이 중 가장 큰 이슈는 `Mock`(또는 다른 `Test Double`)을 사용할 때입니다.

고전적인 `TDD` 스타일은 가능하면 실제 객체를 사용하거나 실제 객체를 사용하기 어색한 경우 다른 `Test Double`을 사용하는 것입니다. 따라서 고전적인 방식으로 위의 예시를 테스트한다면 `Warehouse` 객체는 실제 객체를 사용하고, `MailService`에는 `Test Double`을 사용합니다. 어떤 `Test Double`을 사용하는지는 중요하지 않습니다. 

그러나 `Mock` 방식을 사용할 경우 테스트할 동작을 가진 모든 객체에 대해 항상 `Mock`을 사용합니다. `Warehouse`, `MailService` 모두에 적용됩니다.

다양한 `Mock` 프레임워크가 테스트를 염두해두고 설계되었음에도 많은 고전지향자들은 단지 `Test Double`을 만드는데 유용하다고 생각합니다.

`Mock` 스타일의 중요한 파생물은 `BDD`(Behavior Driven Development, 행동 주도 개발) 입니다. `BDD`는 원래 `TDD`가 설계 기술로 작동하는 방식에 초점을 맞춰 사람들이 `TDD`를 더 잘 배울 수 있도록 돕는 기술입니다. `BDD`는 `Mock` 접근 방식을 취하지만 명명 스타일과 기술 내에서 분석을 통합하려는 욕구 모두를 통해 이를 확장합니다.

## 두 방식 중 선택하기

이 글에서는 상태 검증과 행동 검증, 즉 고전적인 방식과 모의(Mock) 방식의 차이를 설명하였습니다. 그렇다면 이들 사이에서 선택을 할 때 염두해야 할 것은 무엇일까요? 먼저 상태 검증과 행동 검증 선택부터 시작하겠습니다.

가장 먼저 고려해야 할 것은 컨텍스트 입니다. `Order`와 `Warehouse` 처럼 쉬운 관계인지, 아니면 `Order`와 `MailService`처럼 어색한 관계인지 먼저 생각해야 합니다.

쉬운 관계라면 선택은 간단합니다. 고전적인 방식을 사용한다면 `Mock`이나 `Stub`과 같은 어떠한 `Test Double`도 사용하지 않고 실제 객체 및 상태 확인을 사용합니다. 모의 방식으로 사용한다면 행동 검증을 사용하면 되므로 결정을 고민할 필요가 없습니다.

어색한 관계일 경우 모의 방식을 사용할 땐 고민할 필요가 없습니다. 그냥 `Mock`을 사용하고 행동을 검증하면 됩니다. 하지만 고전적인 방식을 선택한다면 어떤 하나의 `Test Double`을 사용하는 것은 큰 문제가 아닙니다. 일반적으로 고전적인 방식은 각 상황에 가장 쉬운 방법을 사용하여 사례별로 결정하게 됩니다.

따라서 상태 검증과 행동 검증은 대부분 어렵게 결정할 사항이 아닙니다. 진짜 문제는 고전적인 방식과 모의 방식을 결정하는 것입니다. 상태 및 행동 검증의 특성이 결정에 영향을 미치므로 여기에 집중할 필요가 있습니다.

하지만 그 전에 극단적인 경우를 생각해보면, 가끔은 어색한 관계가 아니더라도 상태 확인을 사용하기 정말 어려운 일에 직면하게 됩니다. 이것의 좋은 예는 캐시(cache)입니다. 캐시의 요점은 캐시가 hit 할지 missed 일지 여부를 상태에서 알 수 없다는 것입니다. 이는 매우 고전적인 방식에 대해서도 행동 검증이 현명한 선택이 될 수 있는 경우로 두 가지 모두 반드시 예외가 존재합니다.

고전적인 방식과 모의적인 방식의 선택을 고민할 때 고려해야할 요소들을 대략적인 그룹으로 나눴습니다.

### Driving TDD

Mock 객체는 XP(Extreme Programming) 커뮤니티에서 나왔고 XP의 주요 특징 중 하나는 테스트 주도 개발(TDD, Test Driven Development)의 반복을 통해 시스템 설계가 진화하는 것을 강조하는 것입니다.

따라서 모의 방식이 모의 테스트나 설계에 미치는 영향에 대해 이야기하는 것은 놀라운 일이 아닙니다. 특히 그들은 필요 기반 개발(Need Driven Development)이라는 스타일을 옹호합니다. 이 스타일을 사용하면 시스템 외부에 대한 첫 번째 테스트를 작성하고 일부 인터페이스 객체를 SUT로 만들어 사용자 스토리 개발을 시작합니다. Collaborator에 대한 기대치를 고려하여 SUT와 그 이웃 간의 상호 작용을 탐구하여 SUT의 아웃바운드 인터페이스를 효과적으로 설계합니다.

첫 번째 테스트를 실행하면 모의 테스트에 대한 기대치가 다음 단계에 대한 사양과 테스트의 시작점을 제공합니다. 각 기대치를 Collaborator에 대한 테스트로 바꾸고 한 번에 하나의 SUT를 시스템에 적용하는 프로세스를 반복합니다. 이러한 스타일을 outside-in이라고도 하고, 이는 이 스타일을 매우 잘 설명할 수 있는 이름이라고 할 수 있습니다. 계층화 된 시스템에서 잘 작동하는 방식입니다. 먼저 아래쪽 레이어에 있는 모의 레이어를 사용하여 UI를 프로그래밍하는 것으로 시작한다음, 한 번에 한 레이어씩 시스템을 점진적으로 단계별로 실행하면서 하위 계층에 대한 테스트를 작성합니다. 이것은 매우 구조화되고 통제된 접근 방식으로 많은 사람들이 newcomer를 OO(Object-Oriented)와 TDD로 안내하는데 도움이 된다고 생각합니다.

고전적인 TDD는 같은 방침을 제공하지 않습니다. Mock대신 Stub을 사용하여 유사한 단계별 접근 방식을 수행할 수 있습니다. 이를 위해 Collaborator로부터 무언가가 필요할 때마다 테스트에서 SUT가 작동하도록 하는데 필요한 응답을 정확히 하드코등하면 됩니다. 그런 다음 테스트에서 그린라이트를 확인하면 하드 코딩된 응답을 적절한 코드로 교체하면 됩니다.

고전적인 TDD는 다른 작업을 수행할 수도 있습니다. 일반적인 스타일은 middle-out 입니다. 이 스타일에서는 기능을 선택하고 이 기능이 작동하기 위해 도메인에서 필요한 것을 결정합니다. 도메인 객체가 필요한 작업을 수행하도록 하고 작업이 완료되면 맨 위에 UI를 배치합니다. 도메인 로직이 UI로 유출되는 것을 방지하는 도메인 모델에 먼저 집중하기 때문에 많은 사람들이 좋아합니다.

### Fixture Setup

