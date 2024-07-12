# Visibility Modifiers

Kotlin의 Visibility Modifiers에 대한 설명을 정리 해놓았습니다. 공식 문서를 참고하였습니다.

## Modifiers

Kotlin에는 다음과 같은 visibility를 결정하는 modifier가 있다.

- `public`
- `internal`
- `protected`
- `private`

### Packages

Kotlin의 모든 function, properties, classes, objects, interfaces는 package 내부에 바로 만들어질 수 있는데, (즉, 어떤 함수 내부에 선언되는 것이 아니라, 전역처럼 선언이 되는 것.) 우리는 이를 top-level declaration이라고 표현할 것이다.

```kotlin
// file name: example.kt
package foo

fun baz() { ... }
class Bar { ... }
```

각 function, property 등들은 모두 visibility modifier를 적용할 수 있고, 각각의 modifier는 다음과 같은 visibility를 제공한다.

- `public`: visibility modifier를 적용하지 않았을 경우 default로 적용되는 modifier이다. 어디서든 접근이 가능하다.
- `internal`: 같은 module 내에서 누구든지 visible하다.

  > 💫 Module (모듈): Project의 하위 개념으로서, 여러개의 package와 class를 포함하는 개념이다.
  >
  > - Project > Module > Package > Class

- `protected`: top-level declatration 에서는 사용 불가능하다.
- `private`: declaration이 일어난 파일 내부에서만 visibl하다.

기존 Docs에 있는 예시이다.

```kotlin
// file name: example.kt
package foo

private fun foo() { ... } // visible inside example.kt

public var bar: Int = 5 // property is visible everywhere
    private set         // setter is visible only in example.kt

internal val baz = 6    // visible inside the same module
```

### Class

Class의 경우, 다음과 각각의 modifier는 다음과 같은 visibility를 제공한다.

- `public`: 해당 class를 선언하는 그 어떤 존재도 접근할 수 있다.
- `internal`: 이 class를 선언하는 존재 중 같은 module안에 존재하는 존재들만 접근할 수 있다.
- `protected`: 이를 상속하는 class만 접근이 가능할 수 있다.
- `private`: 해당 클래스 내부에서만 접근이 가능하다.

> 💫 `protected`와 `internal` member은 overriding시 따로 visibility를 밝히지 않는다면, 기존의 속성을 그대로 가져가게 된다.

기존 Docs에 있는 예시이다.

```kotlin
open class Outer {
    private val a = 1
    protected open val b = 2
    internal open val c = 3
    val d = 4  // public by default

    protected class Nested {
        public val e: Int = 5
    }
}

class Subclass : Outer() {
    // a is not visible
    // b, c and d are visible
    // Nested and e are visible

    override val b = 5   // 'b' is protected
    override val c = 7   // 'c' is internal
}

class Unrelated(o: Outer) {
    // o.a, o.b are not visible
    // o.c and o.d are visible (same module)
    // Outer.Nested is not visible, and Nested::e is not visible either
}
```

뭐, 당연한 이야기겠지만 Local Declaration의 경우 (Top-level이 아닌 Declaration), visibility modifier를 가질 수 없다.
