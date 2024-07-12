# Inheritance

Kotlin의 Inheritance에 대한 설명을 정리 해놓았습니다. 공식 문서를 참고하였습니다.

## open

`open` keyword는 상속을 위해 기본적으로 필요한 keyword이다. Kotlin에서의 class와 property는 default가 final이기에, `open` 없이는 상속이 불가능하다.

## Inheritance

상속은 다음과 같이 이루어진다.

```kotlin
open class Base(p1: Int){}

class Derived(p: Int) : Base(p) {}
```

상속을 할 시, 상속받는 대상인 Derived Class는 Primary Constructor를 가질 수 있는데, 이럴 경우, Base class는 Drived class의 primary consturtor parameter로 초기화가 되어야 한다.

만약, Derived Class에 primary constructor가 없을 경우, `super` keyword를 통해 Base Class를 initialize 시키거나, initialze를 시켜주는 다른 consturctor에게 위임 (delegation)를 해주어야 한다.

```kotlin
class Derived: Base {
  constructor(p: Int) : super(p)
}
```

상속기 Initialization의 순서는 Base -> Derived이며, 이때 Base에서 Derived에서 override한 요소들을 사용하는 것을 에러를 일으킨다. (이런 일이 일어날거 같지는 않지만, init이나 이 요소를 inherit하는 다른 class에서 무분별하게 사용할 경우 일어날 수 있다.)

- 다음과 같은 이유 때문에, Base Class의 `init` block에서는 `open`을 쓰는 것이 지양된다.

## Overriding

Overriding은 `override` keyword를 사용하여 적용한다.

### Overriding Methods

```kotlin
open class Base {
  open fun a() {/*...*/}
  open fun b() {/*...*/}
}

class Derived(): Base(){
  override fun a() {/*...*/}
  final override fun b() {/*...*/}
}
```

위와 같이 `final` keyword가 붙게 되면 해당 method에 대해서는 이를 상속하는 subclass에서의 re-overriding을 막을 수 있다.

### Overriding Properties

```kotlin
open class Base {
  val prop: Int = 1
}

class Derived(): Base(){
  override val prop: Int = 2
}
```

Property Overriding 또한 method와 같은 방법으로 적용한다. 다만 다음과 같은 특징이 추가된다.

1. `val`로 선언이 된 property에 대해서는 `var`로 overriding이 가능하지만, 그 반대는 불가능하다.

- 이는 `val`은 `get`만을 선언하게 되고, `var`는 추가로 Derived에서 `set`을 정의하기 때문이다.

2. overriding은 primary contructor에서도 적용시킬 수 있다.

```kotlin
open Derived(override var prop: Int = 2): Base(){/* ... */}
```

## Super

Java에서 사용되던 `super` keyword는 여전히 kotlin에서도 유용하다.

```kotlin
//* Kotlin Docs 예시
open class Rectangle {
    open fun draw() { println("Drawing a rectangle") }
    val borderColor: String get() = "black"
}

class FilledRectangle : Rectangle() {
    override fun draw() {
        super.draw()
        println("Filling the rectangle")
    }

    val fillColor: String get() = super.borderColor
}
//* Drawing a rectangle
//* Filling the rectangle
```

Inner class 정의 시, inner class에서 super를 적용해 superclass에 접근하고 싶을 시 그 이름을 super뒤에 @이름으로 적음으로서 적용한다.

```kotlin
//* Kotlin Docs 예시
open class Rectangle {
    open fun draw() { println("Drawing a rectangle") }
    val borderColor: String get() = "black"
}

class FilledRectangle: Rectangle() {
    override fun draw() {
        val filler = Filler()
        filler.drawAndFill()
    }

    inner class Filler {
        fun fill() { println("Filling") }
        fun drawAndFill() {
            super@FilledRectangle.draw() // Calls Rectangle's implementation of draw()
            fill()
            println("Drawn a filled rectangle with color ${super@FilledRectangle.borderColor}") // Uses Rectangle's implementation of borderColor's get()
        }
    }
}

//* Drawing a rectangle
//* Filling
//* Draw a filled rectagle with color black
```

## Inheritance Rules

- Ambiguity Elimination: 여러 클래스를 상속 할 때, 각각의 class들이 같은 이름의 method를 가지고 있는 경우가 있다. 이럴 경우, 이 method를 반드시 override한 후, 본 class만의 implementation을 하거나, 이후에 나올 Generic을 사용하여 `super<Base>`를 적은 후 해당 Base Class의 implementation을 적용시킨다.

## Abstract Class

- `abstract` keyword를 사용하여 Abstract Class를 정의한다. OOP에서 정의하는 추상 클래스와 같이, class의 implementation은 필요하지 않고, `open` keyword 또한 필요하지 않다.

```kotlin
//* Kotlin Docs 예시
abstract class Polygon {
    abstract fun draw()
}

class Rectangle : Polygon() {
    override fun draw() {
        // draw the rectangle
    }
}
```

역도 가능하다. (open class를 abstract class가 override하는 것)

```kotlin
//* Kotlin Docs 예시
open class Polygon {
    open fun draw() {
        // some default polygon drawing method
    }
}

abstract class WildShape : Polygon() {
    // Classes that inherit WildShape need to provide their own
    // draw method instead of using the default on Polygon
    abstract override fun draw()
}
```
