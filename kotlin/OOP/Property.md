# Property

Kotlin의 Class의 Property 대한 설명을 정리 해놓았습니다. 공식 문서를 참고하였습니다.

## Property Declaration

```kotlin
class ExampleClass {
  var a: Int = 1
  val b: String = "Constant!"
}

//...
val exampleClass = ExampleClass()
print("$exampleClass.b")
```

위의 예시와 같이 property는 `var`, 혹은 `val`로 선언을 하며, 해당 property의 이름으로 접근할 수 있다. Kotlin의 property는 **자동으로 getter와 setter를 구현하며**, 나중에 이를 이하와 같은 방법으로 따로 명시적으로 구현할 수 있다.
즉, Kotlin에서의 Property는 다음과 같이 정의할 수 있다.

> 💫 Property: Class의 값을 store하는 Field와 이를 구성하는 getter/setter 모두를 통틀어 칭하는 말

## Getter and Setter

Kotlin의 getter와 setter의 구조는 다음과 같다.

```kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]

val <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>] //* Read Only: not allow a setter
```

이 `get()` `set()`은 class 내의 값을 불러오고 할당할 때 내부적으로 호출된다. 즉, `exmaple.a = 1`이라는 코드는 a의 setter를 호출하여 1을 할당한 것이라고 생각하면 된다.

```kotlin
//* Example
class Rectangle(val width: Int, val height: Int) {
    val area: Int // property type is optional since it can be inferred from the getter's return type => val area get() = this.width * this.height

        get() = this.width * this.height
}

var stringRepresentation: String
    get() = this.toString() //* get() {return this.toString}
    set(value) {
        setDataFromString(value) // parses the string and assigns values to other properties
        //* Convention: setter parameter will be an 'value'
        //* But using another name is still okay.
    }

//* Visibility Change & Annotation for accessor is possible
var setterVisibility: String = "abc"
    private set // the setter is private and has the default implementation

var setterWithAnnotation: Any? = null
    @Inject set // annotate the setter with Inject
```

### Backing Field

Setter에는 조금 더 중요한 이야기가 하나 있다. 다음과 같은 코드가 있다고 가정을 해보겠다.

```kotlin
class Book {
  var title: String
    get() = name
    set(value) {name = value} //* WRONG! name is a property, not a field! - Will cause infinite setter loop
}
```

위에서 이야기를 햇듯, kotlin의 property의 값을 할당하면 setter가 자동적으로 호출되게 된다. 즉, `book.title = "Learning Kotlin"`을 호출하면, `book.title.set("Learning Kotlin")`을 호출한 것과 똑같이 호출된다.
하지만 title은 property이고, 위에서 이야기했듯이, property는 field + getter/setter이다. Property인 title에 값을 할당하는 순간 해당 property는 다시 값을 할당하기 위해 setter를 호출하게 된다.

```kotlin
class Book {
  var title: String
    get() = name
    set(value) {name.set(value)} //* Literally same code with above. Will casue infinite setter loop
}
```

> 💫 Property와 Field를 구분해야지 이해가 되는 개념이다. 즉, 우리는 title라는 field에 값을 할당을 하고 싶지만, 우리가 호출하는 title은 filed가 아닌 property이기 때문에, 값을 할당하는 순간 다시 setter가 호출되게 되는 것이다.

그 결과 setter들이 재귀적으로 호출이 되게 되며, `StackOverflowException`을 발생하게 된다.
Backing Field는 이러한 경우에서 사용할 수 있도록 Kotlin이 제공해주는 기능이다. `field`라는 keyword는 getter/setter내에서 해당 field에 접근할 수 있도록 해준다. 즉, 위의 예시를 우리는 다음과 같이 바꿀 수 있다.

```kotlin
class Book {
  var title: String
    get() = name
    set(value) {field = value} //* Correct!
}
```

### Backing Properties

다음과 같은 코드가 있다고 가정을 해보겠다.

```kotlin
class Example{
  var table: Map<String, String> = null
    private set  //* Will set internally but not allow changes from outside
    get() {
      if (field == null){
        field = HashMap() //Type Paremeters are inferred
      }
      return table ?: throw AssertionError("Set to null by another thread")
    }
}
```

문제가 없는 코드처럼 보인다. 기본적으로 가변적인 table이지만, setter에 `private`을 붙여줌으로서 setter로 통한 변경을 차단한다. 또한 getter를 통해 테이블이 비어있을 경우 새로운 HashMap을 할당하여 이를 반환하고, 아닐 경우에는 기존 데이터를 반환하는 class이다.
하지만 이는 해당 데이터의 성질에 따라 문제가 생길수도 있는데, table의 데이터의 경우, 대부분의 서비스는 해당 데이터를 그대로 보존하여 사용하거나 유저에게 보여지게 하고 싶을 것이다. 하지만 getter는 해당 property의 type을 그대로 따르기 때문에, `var`, 즉 가변적인 상태로 넘겨주게 되어 데이터의 불변성을 보장할 수 없게 된다.

> 💫 즉, 위와 같은 상황에서는 우리는 `var`의 특징을 가진 property를 `val`와 같은 불변적 성격을 가진 데이터로 반환하고 싶다.

다음과 같은 경우, backing property를 사용하여 다음과 같이 해결할 수 있다.

```kotlin
class Example{
  var _table: Map<String, String> = null

  val table: Map<String, Int> //* Backing Property
    get() {
      if (_table_ == null){
        _table = HashMap()
      }
      return _table ?: throw AssertionError("Set to null by another thread")
    }
}
```

이 구조를 통해 데이터 반환시에는 Read-Only로 반환을 할 수 있다. 이 구조를 잘 기억해두도록 하자. 위와 같은 경우, backing field를 사용하지 않아도 되며 (근본적으로는 다른 property이니까!), 구분을 위해서는 기존 property에는 Undersocre `_`를 붙여 구분을 해준다.
