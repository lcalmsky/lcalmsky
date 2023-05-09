## WeakReference란?

`Java`에서 `WeakReference`는 가비지 컬렉터에 의해 강제로 수집될 수 있는 참조를 나타내는 객체입니다. 일반적으로 `Java`에서는 객체에 대한 참조가 있는 경우 해당 객체는 메모리에서 수집되지 않습니다. 하지만 `WeakReference`는 약한 참조를 제공하여 객체가 메모리에서 수집되도록 허용합니다.

`WeakReference`를 사용하면 객체가 더 이상 사용되지 않는 경우 자동으로 메모리에서 제거됩니다. 이는 객체의 수명 주기를 추적하고 메모리 누수를 방지하는 데 유용합니다. 예를 들어 캐시나 캐시 라인에 저장된 객체는 더 이상 필요하지 않을 때 메모리에서 제거되어야 합니다. 이때 `WeakReference`를 사용하면 캐시에서 제거되는 객체의 메모리를 즉시 회수할 수 있습니다.

`WeakReference`는 일반적으로 참조를 저장하기 위해 사용되는 대부분의 객체와 달리, `get()` 메서드를 호출하여 실제 객체를 검색하는 데 사용됩니다. `get()` 메서드를 사용하여 약한 참조가 가리키는 객체를 검색할 수 있습니다. 그러나 `get()` 메서드를 호출하여 반환된 객체가 `null`인 경우, 참조된 객체는 이미 가비지 컬렉터에 의해 수집되었으며 더 이상 사용할 수 없는 상태입니다.

아래는 `WeakReference`를 사용하여 캐시된 객체를 저장하고 조회하는 예제 코드 입니다.

```
import java.lang.ref.WeakReference;
import java.util.HashMap;
import java.util.Map;

public class Cache<T> {
    private final Map<String, WeakReference<T>> cacheMap;

    public Cache() {
        cacheMap = new HashMap<>();
    }

    public T get(String key) {
        T value = null;
        WeakReference<T> weakRef = cacheMap.get(key);
        if (weakRef != null) {
            value = weakRef.get();
            if (value == null) {
                cacheMap.remove(key); // 참조된 객체가 garbage collector에 의해 수집됨
            }
        }
        return value;
    }

    public void put(String key, T value) {
        cacheMap.put(key, new WeakReference<>(value));
    }
}
```

위의 코드에서 `Cache` 클래스는 캐시를 저장하고 검색하는 데 사용됩니다. `put()` 메서드를 사용하여 캐시를 저장하면, 해당 캐시는 `WeakReference`로 래핑되어 `cacheMap`에 저장됩니다. `get()` 메서드를 사용하여 캐시를 검색하면, 해당 캐시의 `WeakReference`를 검색하고 `get()` 메서드를 호출하여 약한 참조가 가리키는 객체를 검색합니다. 만약 해당 객체가 이미 가비지 컬렉터에 의해 수집되었다면, `get()` 메서드는 `null`을 반환하고, 해당 캐시는 `cacheMap`에서 제거됩니다.

## WeakHashMap

`Java`의 `util` 패키지에는 `WeakReference`를 사용해 구현된 클래스가 이미 존재합니다. `WeakHashMap` 클래스는 키와 값 사이의 매핑을 저장하는 해시 테이블 기반의 맵이며, 이 키와 값은 강력한 참조가 아닌 약한 참조로 저장됩니다.

`WeakHashMap` 클래스는 각 항목이 가비지 컬렉터에 의해 수집될 수 있는 약한 참조를 사용하여 구현됩니다. 이로 인해 `WeakHashMap`은 메모리 누수 없이 캐시된 항목을 저장할 수 있습니다. 즉, 객체가 더 이상 사용되지 않으면 가비지 컬렉터에 의해 수집됩니다.

아래는 `WeakHashMap`을 사용한 예시 코드입니다.

```
import java.util.Map;
import java.util.WeakHashMap;

public class Example {
    public static void main(String[] args) {
        Map<Object, String> weakMap = new WeakHashMap<>();
        Object key = new Object();
        weakMap.put(key, "value");
        // key 참조 제거
        key = null;
        // garbage collector 강제 실행
        System.gc();
        // weakMap이 비어있는지 확인
        System.out.println("weakMap.size() = " + weakMap.size());
    }
}
```

위의 코드에서 `WeakHashMap`을 사용하여 키-값 쌍을 저장합니다. key 참조를 제거한 후, `System.gc()`를 호출하여 가비지 컬렉터를 강제 실행하면, 해당 키에 대한 약한 참조가 제거되고, 해당 키와 값에 대한 항목은 `WeakHashMap`에서 자동으로 제거됩니다. 결과적으로, `WeakHashMap`은 비어있는 상태가 됩니다.

> 실행 결과

```
weakMap.size() = 0
```

구현체만 `HashMap`으로 바꿔서 테스트해보면,

```
import java.util.HashMap;
import java.util.Map;

public class Example {
    public static void main(String[] args) {
        Map<Object, String> hashMap = new HashMap<>();
        Object key = new Object();
        hashMap.put(key, "value");
        // key 참조 제거
        key = null;
        // garbage collector 강제 실행
        System.gc();
        // hashMap이 비어있는지 확인
        System.out.println("hashMap.size() = " + hashMap.size());
    }
}
```

> 실행 결과

```
hashMap.size() = 1
```

`key`의 참조를 제거했지만 여전히 메모리를 차지하고 있는 것을 알 수 있습니다.

## Spring에서 사용된 예시

우리가 많이 쓰는 스프링 프레임워크에서도 역시 `WeakReference`를 사용하고 있습니다.

스프링의 `DefaultSingletonBeanRegistry` 클래스는 빈(bean) 객체의 싱글톤 인스턴스를 저장합니다. 이 클래스는 빈(bean) 객체를 저장할 때 `ConcurrentHashMap`을 사용합니다. 이 `ConcurrentHashMap`의 키는 빈(bean) 이름이며, 값은 빈(bean) 객체의 `ObjectFactory`입니다. 이 `ObjectFactory`는 빈(bean) 객체를 생성하는 데 사용됩니다.

스프링은 빈(bean) 객체를 생성할 때, `ObjectFactory`를 사용하여 빈(bean) 객체를 생성하고, 생성된 빈(bean) 객체를 `ConcurrentHashMap`에 캐시합니다. 이때, 캐시에 저장된 빈(bean) 객체의 참조는 `WeakReference`로 저장됩니다.

`DefaultSingletonBeanRegistry` 클래스에서 `ConcurrentHashMap`을 사용하는 이유는 동시성 문제를 해결하기 위해서입니다. 또한, `WeakReference`를 사용하는 이유는 빈(bean) 객체의 메모리 누수를 방지하기 위해서입니다. `WeakReference`를 사용하면 메모리 누수를 방지하면서도 빈(bean) 객체의 캐시를 유지할 수 있습니다.

코드가 길어서 접어두었으니 관심 있는 분은 클릭!

```
public class DefaultSingletonBeanRegistry implements SingletonBeanRegistry {
    // 빈(bean) 객체를 캐시하는 ConcurrentHashMap
    private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

    // 빈(bean) 객체를 생성하는 ObjectFactory를 저장하는 ConcurrentHashMap
    private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

    // 빈(bean) 객체의 약한 참조를 저장하는 ConcurrentHashMap
    private final Map<String, WeakReference<Object>> singletonObjects = new ConcurrentHashMap<>(256);

    public Object getSingleton(String beanName) {
        // singletonObjects에서 빈(bean) 객체의 약한 참조를 가져옴
        WeakReference<Object> singletonObject = this.singletonObjects.get(beanName);
        Object singleton = null;
        if (singletonObject != null) {
            // 약한 참조에서 빈(bean) 객체를 가져옴
            singleton = singletonObject.get();
        }
        if (singleton == null) {
            synchronized (this.singletonObjects) {
                // singletonObjects에서 빈(bean) 객체의 약한 참조를 다시 가져옴
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject != null) {
                    // 약한 참조에서 빈(bean) 객체를 가져옴
                    singleton = singletonObject.get();
                }
                if (singleton == null) {
                    // singletonFactories에서 ObjectFactory를 가져옴
                    ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                    if (singletonFactory != null) {
                        // ObjectFactory를 사용하여 빈(bean) 객체를 생성함
                        singleton = singletonFactory.getObject();
                        // 생성된 빈(bean) 객체를 singletonObjects에 약한 참조로 저장함
                        this.singletonObjects.put(beanName, new WeakReference<>(singleton));
                        // singletonFactories에서 빈(bean) 객체 생성에 사용된 ObjectFactory를 제거함
                        this.singletonFactories.remove(beanName);
                    }
                }
            }
        }
        return singleton;
    }
}
```

### 정리

따라서 `WeakReference`는 다음과 같은 상황에서 사용하면 유용합니다.

1.  캐시 구현: 캐시된 항목에 대한 약한 참조를 사용하여 메모리 누수를 방지하면서 캐시를 구현할 수 있습니다. `WeakHashMap`을 사용하거나 `WeakReference`를 직접 사용하여 캐시를 구현할 수 있습니다.
2.  이미지 캐시: 이미지 캐시에 `WeakReference`를 사용하여 이미지를 캐시할 수 있습니다. 앱에서 이미지를 로드하고 캐시하면, `Bitmap` 객체를 `WeakReference`로 래핑하여 캐시에 저장할 수 있습니다. 이렇게 하면 이미지가 필요하지 않을 때 가비지 컬렉터에 의해 수집됩니다.
3.  콜백 함수: 콜백 함수를 구현할 때, 강력한 참조 대신 `WeakReference`를 사용하여 참조하는 객체를 약하게 참조할 수 있습니다. 이렇게 하면 객체가 필요하지 않을 때 가비지 컬렉터 의해 수집됩니다.
4.  캐싱 레퍼런스: 특정 객체에 대한 캐싱 레퍼런스를 저장할 때, 강력한 참조 대신 `WeakReference`를 사용하여 참조하는 객체를 약하게 참조할 수 있습니다. 이렇게 하면 해당 객체가 필요하지 않을 때 가비지 컬렉터에 의해 수집됩니다.

`WeakReference`는 객체를 가비지 컬렉터에게 수집해달라고 요청할 수 있는 방법이자 메모리를 최적화하는 데 유용한 도구입니다. 그러나 `WeakReference`를 사용할 때는 참조하는 객체가 수집될 수 있다는 점을 고려해야 합니다. 따라서 애플리케이션에서 `WeakReference`를 사용할 때는 신중하게 사용해야 합니다.