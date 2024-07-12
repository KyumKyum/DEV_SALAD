# Class

Kotlin의 Class에 대한 설명을 정리 해놓았습니다. 공식 문서를 참고하였습니다.

## Class Def

Class는 다음과 같이 정의할 수 있다.

```kotlin
class ExampleClass {
 //* Class! 매우 간단간 구조! WA!
}
```

## Constructors (Primary & Secondary Constructors)

Kotlin에는 Primary Constructor와 Secondary Constructor라는 것이 존재한다.

### Primary Consturctor

Primary Constructor는 Class 이름 바로 뒤에 위치하며, Type을 명시할 수 있다.

```kotlin
class ExampleClass (shoutout: String) {  //* init shoutout.
 //*
}
```

Primary Constructor에는 로직을 실행하는 일반적인 코드가 들어갈 수 없다. 해당 코드가 필요한 경우에는 `init` keyword를 통해 만들 수 있다.

```kotlin
class ExampleClass (shoutout: String) {  //* init shoutout.
   init {
       print("Shout out: $shoutout !!!!")
   }
}
```

Property 또한 Primary Constructor에서 정의할 수 있다.

```kotlin
class ExampleClass (val p1: Int, val p2: String, var p3: Boolean = true) {

}
```

다음과 같이 정의할 시 Primary Constructor에서 Property에 대한 declaration & initialization 모두를 할 수 있다.
또한 p3와 같이 default value를 정할 수도 있다.

하지만 Primary Constructor앞에 `constructor` keyword를 사용하여 임을 명시해줘야 하는 경우도 있는데, annotation이 있는 경우나, visibility modifiers가 있을 경우이다.

```kotlin
class ExampleClass @Injection constructor(val p1:Int) {/* ... */}
class ExampleClass private constructor(val p1:Int) {/* ... */}
```

기본적으로 `constructor`는 public이며, 이를 원하지 않을 떄에는 앞에 원하는 visibility modifier를 입력하면 된다.

### Secondary Constructor

Class 내부에도 constructor를 정의할 수 있는데, 이는 Secondary Consturctor로 부른다.

```kotlin
class ExampleClass {
    lateinit val nameLength: Int
    constructor (name: String, age: Int){
        nameLength = name.length()
    }
}
```

위에서 보여지는 것과 같이, Secondary Constructor는 `constructor` keyword를 생략할 수 없다.

또한, secondary constructor와 primary constructor를 같이 사용하는 경우, secondary consturctor의 역할을 primary constructor에 위임 (delegate)해줘야 한다. 이때, `this` keyword를 사용해 위임을 해주며, 이 keyword가 없을 시에는 Compile Error를 띄우게 된다.

```kotlin
class ExampleClass(name: String) {
    lateinit val nameLength: Int
    constructor (name: String, age: Int): this(name){ //* Delegation
        nameLength = name.length()
    }
}
```

`init` 블럭은 primary consturctor에 속한 것이다. 즉, 실행 순서는 primary constructor -> init block -> secondary consturcor이다.
이 순서는 primary constuctor가 없을 경우에도 유효하며 (init block 우선 실행), 이는 delegation이 암묵적으로 일어나기 때문이다.

만약 Primary, Secondary 모두를 생성해주지 않았다면, JAVA에서와 동일하게 Parameter가 없는 Primary Constructor를 형성해준다.

## Class Creation

Kotlin에는 `new` keyword가 **없다.** function call처럼 호출하면 된다.

```kotlin
val user = User()
val user = User('Jay Lim', 23)
```
