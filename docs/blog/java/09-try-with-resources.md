### try-with-resources란?
자바에서 자원을 사용하는 블록을 처리할 때 자동으로 자원을 해제하는데 사용되는 구문으로 사용 방법은 아래와 같습니다.

```java
try (Resource resource = new Resource()) {
    // 자원 사용 코드
} catch (Exception e) {
    // 예외 처리 코드
}
```

이 때 try의 매개변수에서 선언할 수 있으려면 `AutoClosable` 인터페이스를 구현해야 합니다. 대부분의 자바 표준 라이브러리 클래스들은 `AutoClosable` 인터페이스를 구현하고 있습니다. 커스텀 클래스를 사용하려면 해당 클래스가 `AutoClosable` 인터페이스를 구현해야 합니다.

<details>
<summary>AutoClosable 펼쳐 보기</summary>

```java
/*
 * Copyright (c) 2009, 2013, Oracle and/or its affiliates. All rights reserved.
 * DO NOT ALTER OR REMOVE COPYRIGHT NOTICES OR THIS FILE HEADER.
 *
 * This code is free software; you can redistribute it and/or modify it
 * under the terms of the GNU General Public License version 2 only, as
 * published by the Free Software Foundation.  Oracle designates this
 * particular file as subject to the "Classpath" exception as provided
 * by Oracle in the LICENSE file that accompanied this code.
 *
 * This code is distributed in the hope that it will be useful, but WITHOUT
 * ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or
 * FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License
 * version 2 for more details (a copy is included in the LICENSE file that
 * accompanied this code).
 *
 * You should have received a copy of the GNU General Public License version
 * 2 along with this work; if not, write to the Free Software Foundation,
 * Inc., 51 Franklin St, Fifth Floor, Boston, MA 02110-1301 USA.
 *
 * Please contact Oracle, 500 Oracle Parkway, Redwood Shores, CA 94065 USA
 * or visit www.oracle.com if you need additional information or have any
 * questions.
 */

package java.lang;

/**
 * An object that may hold resources (such as file or socket handles)
 * until it is closed. The {@link #close()} method of an {@code AutoCloseable}
 * object is called automatically when exiting a {@code
 * try}-with-resources block for which the object has been declared in
 * the resource specification header. This construction ensures prompt
 * release, avoiding resource exhaustion exceptions and errors that
 * may otherwise occur.
 *
 * @apiNote
 * <p>It is possible, and in fact common, for a base class to
 * implement AutoCloseable even though not all of its subclasses or
 * instances will hold releasable resources.  For code that must operate
 * in complete generality, or when it is known that the {@code AutoCloseable}
 * instance requires resource release, it is recommended to use {@code
 * try}-with-resources constructions. However, when using facilities such as
 * {@link java.util.stream.Stream} that support both I/O-based and
 * non-I/O-based forms, {@code try}-with-resources blocks are in
 * general unnecessary when using non-I/O-based forms.
 *
 * @author Josh Bloch
 * @since 1.7
 */
public interface AutoCloseable {
    /**
     * Closes this resource, relinquishing any underlying resources.
     * This method is invoked automatically on objects managed by the
     * {@code try}-with-resources statement.
     *
     * <p>While this interface method is declared to throw {@code
     * Exception}, implementers are <em>strongly</em> encouraged to
     * declare concrete implementations of the {@code close} method to
     * throw more specific exceptions, or to throw no exception at all
     * if the close operation cannot fail.
     *
     * <p> Cases where the close operation may fail require careful
     * attention by implementers. It is strongly advised to relinquish
     * the underlying resources and to internally <em>mark</em> the
     * resource as closed, prior to throwing the exception. The {@code
     * close} method is unlikely to be invoked more than once and so
     * this ensures that the resources are released in a timely manner.
     * Furthermore it reduces problems that could arise when the resource
     * wraps, or is wrapped, by another resource.
     *
     * <p><em>Implementers of this interface are also strongly advised
     * to not have the {@code close} method throw {@link
     * InterruptedException}.</em>
     *
     * This exception interacts with a thread's interrupted status,
     * and runtime misbehavior is likely to occur if an {@code
     * InterruptedException} is {@linkplain Throwable#addSuppressed
     * suppressed}.
     *
     * More generally, if it would cause problems for an
     * exception to be suppressed, the {@code AutoCloseable.close}
     * method should not throw it.
     *
     * <p>Note that unlike the {@link java.io.Closeable#close close}
     * method of {@link java.io.Closeable}, this {@code close} method
     * is <em>not</em> required to be idempotent.  In other words,
     * calling this {@code close} method more than once may have some
     * visible side effect, unlike {@code Closeable.close} which is
     * required to have no effect if called more than once.
     *
     * However, implementers of this interface are strongly encouraged
     * to make their {@code close} methods idempotent.
     *
     * @throws Exception if this resource cannot be closed
     */
    void close() throws Exception;
}

```

</details>

그리고 전달할 자원은 반드시 `final` 또는 `effectively final`(자바 8 이상에서 도입된 개념으로, 변수가 사실상 final으로 동작하는 경우) 변수여야 합니다. try 블록이 실행되는 동안만 유효해야하기 때문입니다.

마지막으로 자원은 단일 문장으로 초기화되어야 합니다. 

```java
try (String path = "build.gradle";
     BufferedReader bufferedReader = new BufferedReader(new FileReader(path))) {
    return bufferedReader.readLine();
}
```

이런식의 소스 코드는 사용할 수 없고,

```java
try (BufferedReader bufferedReader = new BufferedReader(new FileReader("build.gradle"))) {
    return bufferedReader.readLine();
}
```

이런식으로 한 문장으로 선언해야 합니다.

`try-with-resources` 구문을 사용하면 코드가 더 짧고 분명해지고 만들어지는 예외 정보가 훨씬 유용하다는 장점이 있습니다.

### try-with-resources 구문을 사용했을 때 실제 코드는 어떻게 작성될까?

위에 작성했던 예시 코드

```java
try (Resource resource = new Resource()) {
    // 자원 사용 코드
} catch (Exception e) {
    // 예외 처리 코드
}
```

를 바이트 코드로 변환하면 다음과 같습니다.

```java
Resource resource = new Resource();
Throwable primaryException = null;
try {
    // 자원 사용 코드
} catch (Throwable e) {
    primaryException = e;
    throw e;
} finally {
    if (resource != null) {
        if (primaryException != null) {
            try {
                resource.close();
            } catch (Throwable suppressedException) {
                primaryException.addSuppressed(suppressedException);
            }
        } else {
            resource.close();
        }
    }
}
```

위의 변환된 바이트코드는 `try` 블록에서 자원을 사용하고, `catch` 블록에서 예외를 처리하며, `finally` 블록에서 자원을 해제하는 로직을 보여줍니다. `finally` 블록에서 자원을 해제하는 부분은 `close()` 메서드를 호출하여 자원을 해제하는 코드로 변환됩니다.

특이한 점은 첫 번째 예외가 발생했을 때 `catch` 블럭에서 미리 선언한 변수에 할당하고 다시 던져주고 `finally` 블럭에서 추가로 발생할 수 있는 예외를 첫 번째 예외의 `suppressed`로 등록해준다는 점입니다.

이렇게 처리하면 예외 로그가 보다 더 정확히 남는 것을 확인할 수 있습니다.

> `suppressed`는 주 예외 처리 중에 발생한 부가적인 예외들을 의미합니다.

### 그래서 예외 정보가 어떻게 유용한 걸까?

예시 코드로 확인해보면,

```java
class Resource implements AutoCloseable {
    @Override
    public void close() throws IOException {
        throw new IOException("Closing Resource Failed");
    }

    public void readData() throws IOException {
        throw new IOException("Reading Data Failed");
    }
}

public class TryWithResourceExceptionTest {
    public static void main(String[] args) throws IOException {
        try (Resource resource = new Resource()) {
            resource.readData();
        }
    }
}
```

이 예시를 실행했을 때 다음과 같은 에러 로그를 확인할 수 있습니다.

```text
Exception in thread "main" java.io.IOException: Reading Data Failed
	at Resource.readData(scratch_82.java:16)
	at TryWithResourceExceptionTest.main(scratch_82.java:23)
	Suppressed: java.io.IOException: Closing Resource Failed
		at Resource.close(scratch_82.java:12)
		at TryWithResourceExceptionTest.main(scratch_82.java:22)
```

예외가 발생한 순서대로 예외 로그를 표현하는데, 이후에 발생한 로그가 `Suppressed`에 명시된 것을 확인할 수 있습니다.

`try-catch-finally`에서 처리하게 되면 마지막에 발생한 예외만 표시해주기 때문에 정확히 어떤 부분에서 어떤 에러가 발생했는지 알기 어렵습니다. 특히 `catch` 블럭에서 상세한 예외를 처리하도록 해두었다면 그 외의 에러가 발생하여 제대로 수행되지 않았을 때도 해당 내용을 추적하기 어렵습니다. 이를 방지하기 위해 더 추상화된 레벨의 예외만 처리해도 마찬가지로 정확한 예외를 알기 어렵습니다.

한 줄 요약: try-catch-finally 대신 try-with-resources 씁시다!