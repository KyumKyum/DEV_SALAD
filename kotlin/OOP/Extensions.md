# Extensions

Kotlin의 Extensions에 대한 설명을 정리 해놓았습니다. 공식 문서를 참고하였습니다.
특히 Extension은 기존과 다르게 새로운 내용이 많습니다. 새로운 개념에 주의하며 적도록 하겠습니다.

## Definition

기존의 Decorator라는 것은 "다른 함수를 꾸며주는 역할"로서, 어떤 함수를 받아 명령을 추가하여 함수의 형태로 반환하는 함수를 뜻한다. 어떤 함수의 내부를 수정하지 않고, 기능에 변화를 줄 때 사용을 한다.
Kotlin에서는 이를 대체하는 Extension이라는 것이 존재한다. 이를 활용하면 다음과 같은 엄청난 일도 할 수 있다.

> - Third-party의 library에 있는 class, interface에 새로운 function을 만들 수 있다. (직접 그 Library를 수정하지 않아도!)
> - Third-party의 library에 있는 class, interface에 자신이 원하는 새로운 property 또한 만들 수 있다.

> 💫 쉽게 말해서 기존에 존재하던 Class, Interface에 대한 '확장 정의'라고 생각하자.

이와 같이 선언된 function, property들을 우리는 Extension function / properties 라고 부를 것이다.

## Extension function

Extension Function을 정의하는 방법은 `prefix.funcion()`이며, prefix에는 본인이 extend를 하고자 하는 것의 type (receiver type)을 적으면 된다.
예시를 봐보자.

```kotlin
fun String.sayHi(): String {
  return "Hi! Nice to meet you $this. :)"
}

fun main(args: Array<String>) {
  val name = "Jay"
  print(name.sayHi());
}

// > Hi! Nice to meet you Jay. :)
```

여기서 thi는 extension function 내부에서 받아서 사용하는 receiver object를 의미한다. Docs에서 제공되는 예시를 보고 한번 더 이해해보자.

```kotlin
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}

fun main(args: Array<String>) {
  val list = mutableListOf(1, 2, 3)
  list.swap(0, 2) // 'this' inside 'swap()' will hold the value of 'list'
}
```

Generic을 적용하면 다음과 같이 여러 Type에 대해서도 지원할 수 있다.

```kotlin
fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
```

이렇게 prefix로 붙는 receiver type들은 nullable할 수 있는데 (nullable type), null이 reveiver로 넘어오면 this 또한 null이 된다. 즉, nullable type으로 prefix를 설정하면, null check은 센스가 아닌 필수이다.

```kotlin
fun Any?.toString(): String {
    if (this == null) return "null"
    // After the null check, 'this' is autocast to a non-null type, so the toString() below
    // resolves to the member function of the Any class
    return toString()
}
```

### Important features with extension function

위에서 말했다시피, 이러한 extension function은 새로운 함수를 만드는 것이지, 기존의 것을 수정하는 것이 아니다.

> ❗️ extension function은 기존의 class를 수정하는 것이 아니라, 해당 type에 대해서 .으로 (dot-notation) 호출할 수 있는 function을 만드는 것이다.

또한, extension function은 정적으로 (statically) 처리가 되는데, 이는 함수가 호출될 때, 어떠한 extension function을 호출할지 결정된다는 듯이다. 공식 Docs의 예제를 보며 확인해보자.

```kotlin
open class Shape
class Rectangle: Shape()

fun Shape.getName() = "Shape"
fun Rectangle.getName() = "Rectangle"

fun printClassName(s: Shape) {
    println(s.getName())
}

printClassName(Rectangle())

// > Shape
```

Parameter로 넘겨진 것은 Rectangle이기에, rectangle의 extension function이 호출될 것 같지만, 함수가 호출될 때 Parameter의 type은 Shape이기에, shape의 extension function이 호출되게 된다.

마지막으로, 당연한 이야기지만, 동일한 함수가 맴버 함수로 등록이 되어있는 경우에는 항상 멤버가 이긴다.

```kotlin
class Example {
    fun printFunctionType() { println("Class method") }
}

fun Example.printFunctionType() { println("Extension function") }

Example().printFunctionType()

// > Class Method
```

뭐, 동일한 함수라는 것은 동일한 이름 + 동일한 parameter이니, 동일한 이름이지만 다른 형태의 paramteter의 경우 다른 함수로 인식한다.

```kotlin
class Example {
    fun printFunctionType() { println("Class method") }
}

fun Example.printFunctionType(i: Int) { println("Extension function #$i") }

Example().printFunctionType(1)
// > Extension function #1
```

## Extension properties

위의 기능을 보았으면, extension properties는 어떤 것을 이야기하는지 잘 알것이다. 다음 예시를 봐보자.

```kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1

fun main(args: Array<String>) {
  val list = listOf(1,2,3,4,5)
  print(list.get(list.lastIndex));
}

// > 5
```

이제 List에 대해서 lastIndex라는 extension property를 활용해 마지막 index에 도달할 수 있다.

위에서 이야기한 것과 같이 extension property 역시 기존 class에 새로운 property를 추가시켜주는 것은 아니다. 그러기에 이 extension property를 위한 backing field가 없다. 이 말은 initializer를 extension property에서 사용하지 못한다는 것이고, `val House.number = 1`와 같은 코드는 사용하지 못한다는 것이다. extension property에 값을 할당하거나 가져오는 것은 오로지 getter/setter를 통해서만 가능하다.

## Other Features

이하는 위의 extension function, extension properties를 기반으로 한 kotlin의 extension 관련 추가 기능들에 대해 서술한다.

### Companion object extension

Kotlin에서는 Java의 static의 계보를 잇는 companion이라는 개념이 있다. (사실 JAVA의 static보다는 좀 더 심오한 의미가 있지만, 이에 대해서는 나중에 서술하도록 하겠다.) 이러한 companion object에 대해서도 Companion이라는 prefix를 통해 extension function/properties를 정의할 수 있다.

```kotlin
class MyClass {
    companion object { }  // will be called "Companion"
}

fun MyClass.Companion.printCompanion() { println("companion") }

fun main() {
    MyClass.printCompanion()
}

// > companion
```

### Scope of extension

이러한 extension은 보통 package바로 하단 (즉, top-level에서) 호출하며, 다른 패키지에서 이러한 extension을 사용하고 싶을 때에는 import를 하여 사용한다.

```kotlin
package org.example.declarations

fun List<String>.getLongestString() { /*...*/}
```

```kotlin
package org.example.usage

import org.example.declarations.getLongestString //* Import

fun main() {
    val list = listOf("red", "green", "blue")
    list.getLongestString()
}
```

### Declaring extensions as members

Class 내부에서 다른 class에 대한 extension을 만들 수 있다.

```kotlin
class Host(val hostname: String) {
    fun printHostname() { print(hostname) }
}

class Connection(val host: Host, val port: Int) {
    fun printPort() { print(port) }

    fun Host.printConnectionString() {
        printHostname()   // calls Host.printHostname()
        print(":")
        printPort()   // calls Connection.printPort()
    }

    fun connect() {
        /*...*/
        host.printConnectionString()   // calls the extension function
        //* 보면 알 수 있듯이, 이 extension function은 선언을 한 Connection, extension 대상인 Connection 내부의 Host 모두 접근이 가능하다.

    }
}

fun main() {
    Connection(Host("kotl.in"), 443).connect()
    //Host("kotl.in").printConnectionString()  // error, the extension function is unavailable outside Connection
}

// > kotl.in:443
```

이때, 해당 extension이 호출된 class는 dispatch receiver, 그리고 extension이 만들어져 receiver type으로 호출된 class는 extension receiver라고 한다.
만약 extension reveicer와 dispatch revceiver의 맴버 이름이 같다면 extension receiver가 우선되지만, 이 순서를 바꾸고 싶다면 `this` 문법을 사용하면 된다. (`this`에 대해서는 이후에 다루도록 하겠다.)

```kotlin
class Connection {
    fun Host.getConnectionString() {
        toString()         // calls Host.toString()
        this@Connection.toString()  // calls Connection.toString()
    }
}
```

위와 같은 extension이 member로 선언이 되면, `open` keyword 또한 선언될 수 있는데, 이렇게 되면 이 또한 상속이 되게 된다.

```kotlin
open class Base { }

class Derived : Base() { }

open class BaseCaller {
    open fun Base.printFunctionInfo() {
        println("Base extension function in BaseCaller")
    }

    open fun Derived.printFunctionInfo() {
        println("Derived extension function in BaseCaller")
    }

    fun call(b: Base) {
        b.printFunctionInfo()   // call the extension function
    }
}

class DerivedCaller: BaseCaller() {
    override fun Base.printFunctionInfo() {
        println("Base extension function in DerivedCaller")
    }

    override fun Derived.printFunctionInfo() {
        println("Derived extension function in DerivedCaller")
    }
}

fun main() {
    BaseCaller().call(Base())   // "Base extension function in BaseCaller"
    DerivedCaller().call(Base())  // "Base extension function in DerivedCaller" - dispatch receiver is resolved virtually
    DerivedCaller().call(Derived())  // "Base extension function in DerivedCaller" - extension receiver is resolved statically
}

// > Base extension function in BaseCaller
// > Base extension function in DerivedCaller
// > Base extension function in DerivedCaller
```

위와 같은 코드를 통해, extension receiver는 원래대로 static 하게 처리가 되지만, dispatch receiver는 virtually하게 처리가 됨을 알 수 있다.

### visibility

visibility는 우리가 아는 것과 동일하게 적용이 된다. 일반 function이 visibility scope와 visibility modifier에 영향을 받는것과 동일하게 extension또한 적용된다. 예를 들어,

- top-level에 적용된 extension은 다른 `private` top-level declaration에 접근할 수 있다.
- 해당 receiver type 범위 외에 정의된 extension의 경우, 해당 receiver의 `private` `protected` members에 접근할 수 없다.
