# Null Handling

내가 javascript로 개발을 할 당시, 가장 많이 겪었던 오류가 잡히지 않은 null 값 들이었다. null이 형성되면 안되는 곳에 계속 넘어왔고, 만들고 있는 서비스는 이미 규모가 커져있어 잡기가 힘들었다. 이 모든 것을 graphql로 어느정도 type을 강제했음에도 불구하고 일어나는 일이었다.

  Kotlin은 strict하게 null을 다루며, 이러한 null을 방지할 수 있는 다양한 system을 제공한다. 이에 대해서 사용될 수 있는 몇가지 문법을 서술해보겠다. 

## ? - Nullable

- type뒤에 붙는 ? 는 해당 값이 null이 될 수 있다는 것을 뜻한다.

```kotlin
var str: String? = null //* This value could be nullable
//* Need to null check IOT use this variable as String variable.
if(str != null) return str.length
//* NPE for not having null check

//* This kind of code is not recommended. (Not Safe)

```

## ?. - Null Safe Operator

- 값에 대한 operation을 진행할때 해당 값이 null이면 null을 반환한다. 이는 값의 property에 접근할때에도 사용한다.
- null에 대한 operation은 NPE를 발생시키지만, 해당 Null Safe Operator를 사용하면 null을 반환하는 것으로 safe하게 처리한다. (나중에 try~catch로 해당 오류를 원하는 방향으로 handling 또한 가능)

```kotlin
s?.toUpperCase() //* return null if s is null, not String variable.
user?.name //* return null if user is null.
```

## ?: - Elvis Operator; give default value if null

- 만약 해당 variable이 null이면, 정해진 default 값을 할당하거나 다음에 있는 작업을 처리한다. (개인적으로 정말 마음에 드는 operator이다. 😁😁)

```kotlin
fun getUserName(user:User?) {
  val name: String = user?.name ?: "UNKNOWN"
	//* If user?.name returns null, allocate "UNKNOWN" instead.
}
```

```kotlin
fun getUserName(user:User?) {
  val name: String = user?.name ?: throw NotInitailizedUserError("No Name")
	//* If user?.name returns null, throw new error.
}
```

## !! - Force not null casting

- 해당 variable이 null이 아님을 강제적으로 선언한다. (강제로 not null로 바꾸어준다.)
- !! 선언 이후부터는 null checking과 같이 nullable variable에 필요한 작업이 필요하게 된다.
- 이런 선언 해놓고 null 넣는 배은망덕한 짓거리를 해버리면 NPE가 발생한다. 해당 선언 위치에서 발생하여 debugging할때 훨씬 편하게 할 수 있다.

```kotlin
fun getLength(s: String?) {
  val str: String = s!!
	return s.length
}
```

## let - routine for not null variables

- 해당 variable이 not null인 경우, 해당 객체를 lamda 식 내부로 넘겨서 not null의 경우의 작업을 진행해준다. (이때 내부에는 it으로 해당 data를 받는다.)
- Null이면 lamda 식은 skip된다.
- let에 대해서는 이후 확장함수 (Extension Function)에 대해서 다룰때 더 자세히 설명하겠다.

```kotlin
fun messageEnterPressed (msg: String?) {
  msg?.let {
	  msg -> sendMessage(msg?.sender, msg?.receiver, msg?.payload)
		//* Excecute this lamda expression if msg is not null.
  }
}
```

```kotlin
fun messageEnterPressed (msg: String?) {
  msg?.let {
	  sendMessage(it?.sender, it?.receiver, it?.payload)
		//* Receive target object as 'it'
  }
}
```

## lateint - Declare current var as not-nullable

- lateinit은 현재 선언한 var이 절대로 null이 될 일이 없다는 것을 표현한다. lateinit가 붙은 변수는 첫 상태를 지정할 필요 없이, 선언 이후에 초기화를 할 수 있다.
- 만약 나중에라도 init을 하지 않았을 경우, 컴파일 단계에서 오류를 발생하기 때문에 안전하다. (잠재적 오류, 흔히들 말하는 ‘유령 오류’를 방지한다.)

```kotlin
//* Case 1: not using lateinit
var str:String? = null
//* Not safe (Possible null value at not intended place, especially current str should be not null.)
```

```kotlin
//* Case 2: using lateinit
lateinit var str:String
//... some code ...
cal retCode = "1"
str = "Finished with code $retCode"
//* Assignment after declaration is possible.
//* Exception (Error) for not initalizing lateinit variable at compile time.
```

<aside>
💡 lateinit는 나중에 값이 지정이 되어야 하기 때문에 `const` 에 사용할 수 없고 무조건 `var` 을 사용해야하며, primitive type에는 사용할 수 없다.

</aside>

## by lazy - Define way to initialize when calling

- 현재 해당 변수는 초기화가 되어있지 않지만, 이후에 호출 시에 어떻게 초기화를 할지 결정해준다.
- 현재 초기화를 할 수 있는 방법이 없지만, 이후 필요한 (의존성이 있는) data가 모두 초기화 된 후에 값을 정의할 수 있게 해준다.

```kotlin
lateinit var name: String
val nameLength: Int by lazy { name.length }
//... some code
name = "Jay_Lim"
println("His name is $name and it's length is $nameLength.")
// "His name is Jay_Lim and it's lenght is 7."
```

- lateinit과는 달리 null을 허용해서 초기화가 안되는 것에 대한 exception은 발생하지 않는다. (null)
    - 하지만 위 코드에서는 초기화를 하지 않으면 lateinit에 대한 exception이 발생할 것이다.
- 이후에 값이 정해지만, (즉 초기화가 되면), 이 값은 immutable, 즉 불변성을 보장하는 read-only 값이 된다.