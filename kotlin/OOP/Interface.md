# Interface

Kotlin의 Class의 Interface에 대한 설명을 정리 해놓았습니다. 공식 문서를 참고하였습니다.

## Interface Declaration

Java를 대체하는 언어인 만큼 Interface 구현도 탄탄하게 되어있다.
다중상속이 불가능한 Kotlin과 Java에 다중 상속 기능을 가능하게 만들기 위해 Interface라는 개념이 존재한다. Interface는 다음과 같은 기능을 수행한다.

> 1. 클래스의 기본적인 틀을 제공하며,
> 2. 클래스 간의 중간 매개 역할을 합니다.

Interface는 Abstract Class와는 다르게, Abstract Method와 constants만 포함할 수 있다.

Interface는 `interface` keyword를 사용하여 선언한다.

```kotlin
interface ExampleInterface {
  fun a()
}

class ExampleClass: ExampleInterface {
  override fun a(){
    //* Override
  }
}
```

> 💫 Class는 단 하나의 class만을 상속할 수 있지만, interface는 여러개를 상속할 수 있다!

Docs에서 그대로 가져온 예시이다. 이 예시에서는 Interface 역시 Property를 가질 수 있음을 보여준다. 하지만 interface의 성격으로 인해 기존 class와는 달리 backing field가 존재하지 않고, 오로지 abstract 형태나 accessor의 implementation만이 가능하다.

> 💫 accessor == getter (mutator == setter)

```kotlin
interface MyInterface {
    val prop: Int // abstract

    val propertyWithImplementation: String
        get() = "foo"

    fun foo() {
        print(prop)
    }
}

class Child : MyInterface {
    override val prop: Int = 29
}
```

Interface는 interface를 상속할 수 있다. 지금까지 배운 내용을 종합하면 다음과 같은 코드를 만들 수 있다.

```kotlin
interface Named {
    val name: String
}

interface Person : Named {
    val firstName: String
    val lastName: String

    override val name: String get() = "$firstName $lastName"
}

data class Employee(
    // implementing 'name' is not required
    override val firstName: String,
    override val lastName: String,
    val position: Position
) : Person
```

## Confliction & Ambiguity

여러 Interface를 상속할 수 있는 특징으로 인해, Ambiguity가 발생할 수 있다. 다음과 같은 상황을 생각해보자.

```kotlin
interface A {
    fun foo() { print("A") }
    fun bar()
}

interface B {
    fun foo() { print("B") }
    fun bar() { print("bar") }
}

class D: A, B {
  //...!!
}
```

class D는 A, B에게서 각자만의 방법으로 implement된 `foo()`와 `bar()`를 상속받는다. class D는 저 두 개의 method 모두에 대하여 implementation을 진행해, D가 어떻게 동작을 해야할지 명시해줘 Comfliction을 Resolve해주어야 한다.

```kotlin
class D : A, B {
    override fun foo() {
        super<A>.foo() // A의 foo.
        super<B>.foo() // B의 foo.
    }

    override fun bar() {
        super<B>.bar() // B의 bar
    }
}
```

## Functional (SAM - Single Abstract Method) Interface

Interface에도 특별한 interface가 있다. 바로 functional, 혹은 SAM (Single Abstract Method) interfac이다. 이름 그대로, 단 한개의 abstract method을 가지고 있는 interface가 이에 해당한다.
Functional Interface는 `fun` keyword를 적어서 사용한다.

```kotlin
fun interface ExampleSAMInterface {
   fun a()
}
```

도대체 어디서 이것을 사용하는가! Kotlin의 feature중 하나인 Lambda Expression와 함께라면 기가막힌 코드를 짤 수 있다.

예를 들어, Int를 parameter로 받아, 원하는 조건을 판단한 뒤, Boolean 결과를 return하는 IntPredicate라는 SAM Interface가 있다고 해보자.

```kotlin
fun interface IntPredicate {
   fun accept(i: Int): Boolean
}
```

이 `IntPredicate`를 짝수인지 아닌지를 판단하는 데 사용하고 싶다. 이를 그냥 사용하면 다음과 같은 코드가 된다.

```kotlin
val isEven = object : IntPredicate {
   override fun accept(i: Int): Boolean { //* Override function
       return i % 2 == 0
   }
}
```

하지만 Lambda Expression와 같이하면 다음과 같이 표현도 가능하다.

```kotlin
// Creating an instance using lambda
val isEven = IntPredicate { it % 2 == 0 }
//* lambda로 인자를 it으로 받는다. 그 결과를 Boolean으로 return. 이것은 IntPredicate이 method가 하나밖에 없는 SAM Interface이기 때문이다.
```

즉, SAM Interface와 Lambda Expression을 적절히 사용하면, 간결한 코드를 만들 수 있다.

```kotlin
fun interface IntPredicate {
   fun accept(i: Int): Boolean
}

val isEven = IntPredicate { it % 2 == 0 }

fun main() {
   println("Is 7 even? - ${isEven.accept(7)}")
}

//* Is 7 even? - false
```

### Type Alias

위 코드를 `typealias` keyword를 사용해서도 만들 수 있다.

```kotlin
typealias IntPredicate = (i: Int) -> Boolean

val isEven: IntPredicate = { it % 2 == 0 }

fun main() {
   println("Is 7 even? - ${isEven(7)}")
}

//* Is 7 even? - false
```

하지만 분명 functional interface와 typealias는 다른 것인데, 그 차이는 다음과 같다.

- Type Alias는 기본적으로 존재하는 type에 대한 새로운 이름이며 (즉, 새로운 type를 만드는 것이 아니다.), 단 한개의 member로 이루어져있다.

  > ✔️ 만약 서비스가 기존에 존재하는 parameter / return type를 필요오 하면, 단순 functional type, 혹은 type alias를 사용하면 된다.

- Functional Interface는 새로운 type를 만들며 (새로운 Interface 형성 -> 기존의 type과 function이 수행할 수 없는 기능을 수행), 여러개의 non-abstract member를 포함한다. (한 개의 abstract method만 존재하면 된다.) 즉, type alias보다 더욱 flexible 하지만, costly하다.
  > ✔️ 만약 서비스가 기존의 functional type으로 표현될 수 없는 복잡한 entity를 요구한다면, 이는 이를 위한 개별적인 functional interface가 필요한 일이다.
