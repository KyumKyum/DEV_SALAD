# Data Classes

Kotlin의 특별한 class인 Data Class와 Sealed Class에 대한 설명을 정리해놓았습니다. 공식 문서를 참고하였습니다.

## Data Class

어떤 한 클래스를 선언하는데에 있어 그 목적은 여러가지이고, 그 중 많은 경우는 Data를 담기 위한 곳이다. Kotlin은 이러한 data를 담는 class를 위해 `data`라는 키워드를 만들었다.

```kotlin
data class User(val name: String, val age: Int)
```

이와 같이 data class는 기존 class에서 추가적인 기능을 제공해주는데, 가령 primary constructor에 정의된 모든 properties에 대해서

- `equals()` / `hashCode()`
- `toString()` (위와 같은 예시에서는 "User(name=$name, age=$age)"가 나온다.)
- `componentN()` N은 각 property의 declaration 순서에 따른다.
  > 💫 val (name, age) = user와 같은 desctructuring을 진행할 시,
  >
  > - val name = user.component1()
  > - val age = user.component2()와 같이 표현할 수 있다.
- `copy()`
  와 같은 member들을 자동으로 만들어준다.

물론 위와 같이 제공되는 member를 100% 활용하기 위해서는 data class 또한 다음과 같은 조건을 충족해야 하는데,

1. primary constructor에서 적어도 하나 이상의 parameter를 가져야 하고
2. priamry constructor의 모든 parameter는 `val` 혹은 `var`로 명시가 되어야하며,
3. `abstract`, `open`, `sealed`, `inner`와 같은 keyword가 붙으면 안된다. (데이터를 의미하는 class가 이러한 정보가 있으면 안된다는건 자명한 사실!)

또한, 위와 같이 만들어진 data class는 다음과 같은 규칙을 따른다.

1. 위와 같이 만들어진 member 중 `equals()`, `hashCode()`, `toString()`의 경우, 같은 이름의 맴버기 이미 존재하면 (혹은 inherited된 member 중에서 `final`로 선언되어 overriding이 불가능하면), 위 함수들은 만들어지지 않으며, 기존의 implementation이 유지된다.
2. Inherited된 member 중에서 `componentN()`이라는 member가 있으면 (Superclass와 해당 method가 `open`되어있다고 가정), 이 memeber는 overriding이 진행되어 해당 child class에 형성이 된다. 위 상황의 경우, 해당 memeber가 `final`로 선언되어 있을 경우 오류가 난다.
3. `componentN()`과 `copy()`에 대한 explicit implemetation은 불가능하다.
   > 💫 Explicit Implementation: Interface를 상속 받을 때, 해당 interface의 이름을 명시하여 method를 implement하는 경우이다. 즉 A라는 Interface를 상속받을 때, 해당 A가 copy() method를 가지고 있으면, `A.copy()`와 같은 implementation은 안된다는 것이다.

그리고 위에서 명시하였듯이, 위 member들은 오로지 primary constructor에서 선언된 property에 대해서만 제공을 해준다. 다음 클래스를 예로 들자면,

```kotlin
data class User(val name: String) {
    var age: Int = 0
}
```

`val name`에 대해서는 `equals()`, `hashCode()`, `toString()`이 만들어질 것이고, `User` class는 `name`을 반환하는 `component1()`만 갖게 된다.

```kotlin
val user1 = User("John")
val user2 = User("John")
user1.age = 10
user2.age = 20
```

위와 같은 경우에도 user1과 user2는 동일한 것으로 취급될 것이다. (data class의 대상인 `name`이 동일하기 때문!)

`copy()`또한 다음과 같이 이루어진다.

```kotlin
data class User(val name: String, val age: Int)

val jack = User(name = "Jack", age = 1)
val olderJack = jack.copy(age = 2)
```

이와 같이 일부분을 변경시키고, 나머지를 변경되지 않은 상태로 copy를 할 수 있다.

## Do not use data class as JPA Entity!

Java 시절부터 아주 긴히 내려오는 ORM이다. 여기서 각 Entity들에 대한 정보를 class로 만들어두는데, 이 경우에 '어? 데이터 관련 클래스니까 `data`를 붙여도 되는건가?' 싶을수도 있다. 하지만 JPA Entity로 class를 사용할 때에는 `data` keyword를 쓰지 않는 것이 좋다.
그 이유는 다음과 같다.

1. Spring에서도 data class 사용을 지양하고 있다. 이는 JPA가 data class가 만들어주는 여러 member 들을 염두하지 않고 만들어졌기 때문이다. 뭐, 굳이 사용하려면 저 member 변수들을 override하는 방법이 있긴 한데, 굳이?
2. data class는 `open` 을 사용할 수 없어 상속이 불가능한데, 그러면 상속이 필요없는 데이터는 사용해도 되지 않느냐! 하는데 Spring에서 사용하는 `allopen` plugin은 class, data class 상관없이 모두 open을 붙인다(ㅋ...) 다음과 같은 이벤트는 data class의 의도와 spec을 무시하는 결과를 낳게 된다.
3. JPA는 transaction내 entity에 대한 상태 이상이 발견이 되면, `save()`와 같은 repository method를 호출하지 않아도 바로 update를 반영해준다. (Entity Dirty Check) 기존의 data class의 경우, `var`로 선언된 field에 대해 상태 변경을 하면 update가 일어나지만, 만약 field가 `val`로 선언이 되어 있어 `copy()`를 이용하여 상태 변경을 시도하면, JPA 입장에서는 field가 변경된 것이 아니라 object가 새로 형성된 것이니 update를 하지 않는다. 이러한 경우에는 따로 `save()`를 호출해야한다.

즉, Entity class에 대해서는 그냥 class를 쓰는 것이 훨씬 이득이다!
