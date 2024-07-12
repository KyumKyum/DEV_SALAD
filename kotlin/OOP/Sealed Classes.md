# Sealed Classes

Kotlin의 Sealed Classes에 대한 설명을 정리 해놓았습니다. 공식 문서를 참고하였습니다.
Sealed Class는 Kotlin에서는 새로운 개념입니다. 이점 참고하고 잘 읽어주시면 감사하겠습니다.
이번 글을 작성하는 데에는 Docs 이외에 추가 레퍼런스 자료를 확인했습니다. 이는 하단에 Reference에 출처를 남겨놓겠습니다.

## What is Sealed Class?

일단 다음 코드를 살펴보자. 아주 간단한 RPG게임이 있고, 워리어, 메이지, 로그라는 세가지 직업군이 RPGClass()라는 abstract class를 상속받는다고 가정해보자.

```kotlin
abstract class RPGClass

class Warrior: RPGClass()
class Mage: RPGClass()
class Rogue: RPGClass()

//...

fun activateSkill(rpgClass: RPGclass): String {
  return when (rpgClass) {
    is Warrior -> "Berserk Mode!"
    is Mage -> "Fireball!"
    is Rogue -> "Assasinate!"
  }
}

// Error! 'when' expression must be exhaustive, add necessary 'else' branch
```

위와 같은 오류 메세지를 토해내며 해당 코드는 컴파일이 실패한다. 이 이유는 Compiler는 해당 RPGClass()를 누가 상속받는지, 즉, 해당 클래스를 상속받는 하위 클래스의 종류를 알지 못해서 일어나는 일이다.

그럼 `else `를 추가하면 되는 일인가? 물론 그렇게 해도 해결이 된다. 하지만 `else`가 추가되는 순간 다음과 같은 코드도 모두 컴파일이 성공적이게 된다.

```kotlin
fun activateSkill(rpgClass: RPGclass): String {
  return when (rpgClass) {
    is Warrior -> "Berserk Mode!"
    is Mage -> "Fireball!"
    is Rogue -> "Assasinate!"
    else -> "Unknown!"
  }
}

fun activateSkill(rpgClass: RPGclass): String {
  return when (rpgClass) {
    is Warrior -> "Berserk Mode!"
    is Mage -> "Fireball!"
    else -> "Unknown!"
  }
}

fun activateSkill(rpgClass: RPGclass): String {
  return when (rpgClass) {
    is Warrior -> "Berserk Mode!"
    else -> "Unknown!"
  }
}
```

`else`는 위에서 모든 케이스가 속하지 않으면 최종적으로 다다르는 default이기 때문이다. 위와 같은 행위는 특정 케이스에 대한 처리가 되지 않고, 저렇게 잘못 적힌 코드가 있어도 오류가 없으니 찾기도 힘들 것이다.

여기서 Sealed Class가 등장한다. Sealed class는 조금 특별한 abstract class로써, Sealed Class는 chiled class의 종류를 제한시켜 Compiler가 해당 sealed class의 자식 class가 어떤 것이 있는지 알 수 있게 할 수 있다.

> 💫 Sealed Class는 Abstract Class가 상속받는 Derived Class의 종류를 제한하는 특성을 가진다.

다음 코드를 봐보자.

```kotlin
sealed class RPGClass()

class Warrior: RPGClass()
class Mage: RPGClass()
class Rogue: RPGClass()

//...

fun activateSkill(rpgClass: RPGclass): String {
  return when (rpgClass) {
    is Warrior -> "Berserk Mode!"
    is Mage -> "Fireball!"
    is Rogue -> "Assasinate!"
  }
}
```

이제 이 코드는 문제가 없다. 이제 Compiler는 sealed class인 RPGClass를 상속하는 class의 종류를 모두 알기 때문이다. `else`없이도 이제 모든 종류에 대한 조건 처리를 할 수 있다.

```kotlin
sealed class RPGClass()

class Warrior: RPGClass()
class Mage: RPGClass()
class Rogue: RPGClass()
class Druid: RPGClass() //* Newly Created

//...

fun activateSkill(rpgClass: RPGclass): String {
  return when (rpgClass) {
    is Warrior -> "Berserk Mode!"
    is Mage -> "Fireball!"
    is Rogue -> "Assasinate!"
  }
}
//* Error!
```

물론 이제 Compiler는 상속하는 class의 종류를 모두 알기 때문에, 만약에 처리가 안된 조건이 존재할 시 오류를 일으킨다. 단순히 `else`를 사용하기 보다는, 이에 대한 처리를 모두 해주는 것이 바람직하며, sealed class의 목적에도 맞다.

```kotlin
fun activateSkill(rpgClass: RPGclass): String {
  return when (rpgClass) {
    is Warrior -> "Berserk Mode!"
    is Mage -> "Fireball!"
    is Rogue -> "Assasinate!"
    is Druid -> "Fourteen!"
  }
}
```

## Keyword `sealed`

`sealed`는 비단 `class`에만 붙을 수 있는 것이 아니고, 또 class만 상속받을 수 있는 것이 아니다.

```kotlin
sealed interface Error

sealed class IOError(): Error

class FileReadError(val file: File): IOError()
class DatabaseError(val source: DataSource): IOError()

object RuntimeError : Error
```

여기서 눈에 띄는 것이 바로 객체, `object`로 상속을 받는 것이다. sealed class를 class로 상속받는 경우는 다음과 같다.

- 즉 sealed class에 state variale/object가 있거나
- equals()를 override해야하는 경우

이외의 경우에는 메모리 절약을 위해 Singleton 패턴 (class 하나에 하나의 object instance만을 가지고 있는 패턴)으로 메모리에 한번만 올리는 `object`에 상속하는 것이 더욱 낫다.

## Difference with enum class

`enum` class와 비슷하다는 이야기도 듣는다.

```kotlin
enum class Direction {
    NORTH, SOUTH, WEST, EAST
}

enum class Color(val rgb: Int) {
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF)
}
```

다음과 같이 (물론 consturctor가 동일해야 한다는 제약이 있지만), sealed class와 같이 종류를 제한할 수 있고, sealed class가 할 수 없는 iteration 또한 가능하다는 것은 강점이다.

하지만 sealed class는 위의 특징 말고도 더 중요한 특성이 있으며, 사실 이 특성이 해당 종류의 클래스 이름이 왜 `sealed`로 붙게 되었는지 설명해준다.

## Characteristic of `Sealed`

**1. 같은 패키지의 자식 클래스만 상속이 가능하다.**

- 만일 다른 패키지에서 sealed class를 상속하려고 하면 오류가 나온다. 오직 같은 패키지에서만 상속이 일어난다. 외부의 class에서 접근하거나 상속하여 사용할 수 없다.
- 정말 중요한 특성인데, 이 특성을 바탕으로 외부 라이브러리 API에서 사용하는 class, interface를 client가 함부로 상속받을 수 없게 만드는 것이다.
  - 만에 하나 외부 라이브러리가 사용하는 Error class를 상속받아 해당 클래스를 extend/implement하는데에 아무런 제약이 없다면, 해당 client에 의해 새로 정이된 Error에 대해서는 library가 제대로 handling할 수 없게 된다.
  - `sealed` keyword를 통해, 본인들이 정하고자 하는 종류를 명시하여 개발을 진행할 수 있고, 이 이후에는 '밀봉'하여 이를 상속하는 다른 종류를 만들지 못하게 하는 것. 이가 `sealed`가 사용되는 진짜 이유라고 할 수 있겠다.

> 💫 즉, `sealed` keyword는 외부에서는 공개가 되지 않은 상태, 즉 '밀봉'된 상태를 유지하면서, 본인의 자식 클래스들에게는 상속을 진행하는 사용이 가능하게 만들어준다.

2. Sealed Class는 인스턴스화 될 수 없다.

- abstract class의 역할을 일부 하고 있으나, sealed class 직접 객체를 만들 수는 없다. 뭐, 다음과 같은 것은 불가능 하다는 것이다.

```kotlin
fun impossibleTrial(){
  RPGClass()
}
//* Error: Sealed types cannot be instantiated.
```

##### References

[[Kotlin] Kotlin sealed class란 무엇인가?](https://kotlinworld.com/165)
