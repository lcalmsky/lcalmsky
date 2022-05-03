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

### EasyMock 사용


