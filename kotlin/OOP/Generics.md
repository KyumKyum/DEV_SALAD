# Generics

Kotlin의 Generics에 대한 설명을 정리 해놓았습니다. 공식 문서를 참고하였습니다.
Generic은 OOP에서 매우 중요한 개념이지만, 난이도가 하드코어하니 정신 바짝 차리자!
나 또한 계속 업데이트 할 것이고, 아직 inline function의 `reified` keyword를 완전히 이해하지 못해, 본 내용도 80%만 정리가 되어 있는 상태이다. 꾸준히 공부하고 업데이트를 해야겠다.

## What is generic?

Kotlin의 Generic은 Java에서와 마찬가지로 type을 결정해주는 type parameter이다.

```kotlin
class Box<T>(t: T) {
    var value = t
}

val box: Box<Int> = Box<Int>(1)

val boxInferred = Box(1) // 1 has type Int, so the compiler figures out that it is Box<Int>
```

위와 같이 type parameter에 따라 class property와 member의 type 또한 결정이 되고, 이는 inferred될 수 있다.

## Variance

Java에서 편리하면서도 가장 빡쳤던 문법이 바로 wildcard 문법일 것이다.

> 💫 아는 사람들은 다 아는 `<?>`... 남용되는 순간 그 코드는 겉잡을 수 없게 되는 것..

Kotlin에는 이러한 wildcard 문법이 없는 대신, declaration-site varicance와 type projection이라는 개념이 등장한다.

우선, Kotlin의 variance를 설명하기 전, Java의 Wildcard의 개념에 대해서 먼저 알아두는게 좋을 것 같다. 잠깐 Java 이야기를 시작하겠다.

### Generic; Java

Java에서의 Generic 개념이다. 본 Kotlin의 Generic을 이해하기 위해 필수적인 내용들이 들어가 있으니 모두 읽어보도록 하자.

#### 1. 하위 타입 (Subtype)

하위 타입은 모든 것을 class로 보는 Java에서 시작되는 개념으로서, 하위 타입이 상위 타입을 상속한다는 개념이다.

```Java
class String extends Object {/*...*/}
```

> 💫 하위 타입: type A의 값이 필요한 곳에 type B의 값을 넣어도 아무런 문제가 없다면 B는 A의 하위 타입이다.

즉, 다음과 같은 관계가 성립이 된다.

a. 이하는 하위 타입 -> 상위 타입의 관계이다 (B -> A)

```plaintext
- Integer -> Number
- Integer -> Integer?
- String -> Object
```

b. 이하는 하위 타입 -> 상위 타입의 관계가 아니다 (B -X-> A)

```plaintext
- Integer -X-> String
- Integer? -X-> Integer
- List<Integer> -X-> List<Number> (이유는 하단에 설명)
```

#### 2. Covariant

Covariant의 개념을 정리하고 가겠다. 이 개념은 앞으로도 계속 나올 개념이니, 확실히 이해하도록 하자.

- Covariant: 어떠한 type A가 있을 때, 이 type을 부모 객체의 type으로 변환시킬 수 있는 경우.

```plaintext
Integer, Long, Double -> Number -> Object
String -> Object
```

#### 3. Invariant

```java
class Integer extends Number {...}
```

위에서 Integer는 Number를 상속하는 객체고 Integer은 Number의 하위 타입이다.
하지만 이러한 상속관계는 generic에서 이루어지지 않는다.

> 💫 Integer은 Number의 subtype이지만, `List<Integer>`은 `List<Number>`의 subtype이 아닐 뿐더러, 관련이 없는 타입으로 취급한다.

즉, **Java의 Generic type은 invariant이다.** 이것이 왜 `List<Integer>`이 `List<Number>`의 subtype이 아닌지에 대한 이유이다.

```java
void integerArray() {
  List<Integer> integerList = Arrays.asList(0,1,2);
  print(integerList); //* Error!!!
}

void print(Collection<Number> c){
  for(Number e : c){
    System.out.print(e);
  }
}
```

##### Generic에서 상속 관계를 인정해주지 않는 이유?

Generic에서 상속 관계를 인정해주지 않는 이유는 '안정성'이다.
예를 들어 위의 예시에서, `List<Integer>`이 `List<Number>`의 subtype임을 인정해준다면, `List<Double>`도 subtype임이 인정이 될 것이다.
하지만, 문제는 Generic이 일어나는 함수 내부에서는 넘어오는 것의 type이 `Number`인지, `Integer`인지, `Double`인지 알 길이 없다.

> 💫 만약 넘어온 type은 `Integer`인데, Double만 가능한 연산을 한다면 원하지 않는 결과를 내올 것이며, `Number`는 없는 `Integer`의 연산을 진행하면 오류가 날 것이다.

Number관련 연산이어서 예시가 단순하지, 만약에 모든 type으로 확장되면 Java에서 가장 많이 볼 수 있는 오류 중 하나는 `ClassCastException`와 같은 type관연 오류일 것이다.

위와 같은 이유로 해당 코드는 Compile Error가 나오게 된다. Generic으로 만들어진 Object나 Collection은 invariant이기 때문에, Java Compiler의 입장에서는 전혀 다른 type이 parameter로 넘어온 것으로 판단한 것이다.

#### 4. Wildcard

그래서 `?`라는 Wildcard 개념이 나오게 되었다. 위 코드를 다음과 같이 바꾸면 오류가 사라진다.

```java
void integerArray() {
  List<Integer> integerList = Arrays.asList(0,1,2);
  print(integerList);
}

void print(Collection<?> c){
  for(Object e : c){
    System.out.print(e);
  }
}
```

#### 5. Bounded Wildcard

Wildcard는 '모든 타입을 대신할 수 있는 unknown type'이다. 물론, 모든 타입을 대신하기 때문에 남용이 되면 여러 문제를 일으킬 수 있다. 그래서 다음과 같이 범위를 제한시켜 놓은 bounded wildcard를 사용한다.

##### 5-1. `? extends T`; Covariance

Integer은 Number의 하위 타입이지만, `List<Integer>`와 `List<Number>`는 incovariant이기에, wildcard를 사용해야했었다.

`? extends T`의 역할은 일종의 업캐스팅 (UpCasting)인데, 즉, `T`나 `T`를 상속하는 모든 child class들을 모두 `T`로 캐스팅하여 받는 것이다.

```java
void integerArray() {
  List<Integer> integerList = Arrays.asList(0,1,2);
  print(integerList); //* ERROR!
}

void print(Collection<? extends Number> c){
  for(Number e : c){ //* Access as number
    System.out.print(e);
  }
}
```

엇 그럼 이런 문제가 생길수도 있다. 예를 들어, Integer는 가능한데, Number는 못하는 연산이 있을수도 있다. 또한 Integer와 Double 모두 Number를 상속받는 child class인데, 이 둘이 할 수 있는 연산은 서로 다르다.

그래서 `? extends T`는 **Read-Only 즉, 데이터를 읽는 것은 가능하되, 쓰는 것은 불가능하다.** 위의 코드에서 만약에 데이터를 적으려고 하면 다음과 같은 오류가 발생할 것이다.

```java
void addThree(Collection<? extends Number> c){
  for(Number e : c){ //* Access as number
    e += 3 //!!!! ERROR: expected ? extends number, but int.
  }
}
```

`? extends T`는 데이터를 제공하는 역할만 한다고 해서 **Producer**라는 이름으로도 불리게 된다.

Generic의 Covariance는 다음과 같이 정리할 수 있겠다.

> 1. Generic의 bounded wildcard의 사용법 중 `? extends T`라는 활용은 covariance를 적용 시키는 방법으로 wildcard의 제한을 둔다.
> 2. `?`에는 `T`, 혹은 `T`를 상속하는 child class만이 올 수 있고, 이들 모두 `T`로 접근을 한다.
> 3. 다만, child class마다 어떤 차이점이 있는지는 compiler가 알지 못하니, 오로지 Read-Only로만 데이터를 제공한다.
> 4. 데이터를 제공하는 역할이기에 Producer라는 이름으로 불린다.

#### 5-2 `? super T`; Contravariance

문제는, 사용자는 Read-Only 데이터만 원하지 않는다는 것이다. 사용자는 데이터를 쓰기를 원할 수 있다. 이때 사용할 수 있는 것이 바로 `? super T`이다.

`? super T`는 다운캐스팅 (DownCasting)을 진행한다. 즉, `?`에 오는 type은 `T`나, `T`가 상속하는 `T`의 부모 클래스 (super class)들이다.

위의 `addThree()`를 다음과 같이 바꾸면 오류가 사라지게 된다.

```java
void addThree(Collection<? super Integer> c){
  for(Integer e : c){ //* Access as Integer
    e += 3
  }
}
```

이 경우는 Write 작업 시에도 오류날 가능성이 없다. Child class(Integer)는 Super Class(Number)의 모든 것을 가지고 있고, 거기에 부가적인 요소가 추가된 것이다. 즉, Child Class는 Parent Class의 모든 methoc와 그 이상을 가지고 있다는 것이다.

다시 말하면, 부모 클래스는 자식이 가지고 있는것을 가지고 있지 않아, 값에 접근할 때 오류를 일으킬 가능성이 있다. 그래서 `? extends T`와 반대로 `? super T`는, **Write-Only 즉, 데이터를 쓰는 것은 가능하되, 읽는 것은 불가능하다.**

`? super T`는 데이터를 사용하는 역할만 한다고 해서 **Consumer**라는 이름으로도 불리게 된다. 이러한 개념을 Covariant와 반대된다고 하여 Contravariant라고 이야기 한다.

Generic의 Contravariant는 다음과 같이 정리할 수 있겠다.

> 1. Generic의 bounded wildcard의 사용법 중 `? super T`라는 활용은 covariance의 반대적 개념을 적용 시키는 방법으로 wildcard의 제한을 둔다.
> 2. `?`에는 `T`, 혹은 `T`가 상속하는 super (Parent) class만이 올 수 있고, 이들 모두 `T`로 접근을 한다.
> 3. 다만, Super Class는 Child class가 가지고 있는 member가 없을수도 있으니, 오로지 Write-Only로만 데이터를 제공한다.
> 4. 데이터를 사용하는 역할이기에 Consumer라는 이름으로 불린다.

해당 내용에 대한 이해가 더 필요하면 하단에 있는 Reference를 참고하자.

### Kotlin - Declaration-site variance; `out`, `in`

자, 이제 다시 Kotlin으로 돌아오자.
위의 내용을 어느정도 이해했다면, JAVA에서 다음과 같은 코드는 Error를 일으킨 다는 것을 알 수 있다.

```java
interface Source<T> {
    T nextT();
}

void demo(Source<String> strs) {
    Source<Object> objects = strs; // !!! Not allowed in Java
    // Invariant Generic
}

//* Resolved by...
void demo(Source<String> strs) {
    Source<? extends Object> objects = strs; // !!! Not allowed in Java
    // Invariant Generic
}
```

Resolved by...하단에 있는 코드를 사용하면 해결이 되지만, 너무 의미없고 (개인 의견이 아니라 공식 Docs에서도 meaningless라고 나와있을 정도로 의미없다!) 난해하다. (저자가 Java를 싫어하게 된 이유이기도 하다.)

Kotlin은 다음과 같은 간단한 variance keyword로 해결한다.

`covariance: ? extends E => out`
`contravariance: ? super E => in`

즉, 위의 코드를 코드는 Kotlin에서 다음과 같이 작성되면 오류가 발생하지 않는다.

```kotlin
interface Source<out T> {
    fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs // This is OK, since T is an out-parameter
    // 위와 같이 적었을 때 `Source<? extends Object>`와 같게 사용할 수 있다.
}
```

> 💫 General Rule:
>
> - class `C`에 대해서 type parameter `T`를 받을 때, variance annotation인 `out`이 붙으면, `C`는 `T`의 Producer (Covariant)가 되게 된다.
> - class `C`에 대해서 type parameter `T`를 받을 때, variance annotation인 `in`이 붙으면, `C`는 `T`의 Consumer (Contravariant)가 되게 된다.

```kotlin
interface Comparable<in T> {
    operator fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0) // 1.0 has type Double, which is a subtype of Number
    // Thus, you can assign x to a variable of type Comparable<Double>
    val y: Comparable<Double> = x // OK!
}
```

## Type Projection

위에서 `in`, `out` keyword를 활용함으로서 contravariant, covariant를 만들어줌으로써 subtype문제를 해결하였다. 이러한 문법은 각 type의 상위, 혹은 하위 집합을 정해줌으로서 한가지 type인 `T`로 접근할 수 있게 함으로써 쉽게 해결을 하였다.
하지만 어떤 특정한 class는 `T` 하나만 return할 수 있도록 제한할 수 없는 경우도 있다. 가장 대표적인 예시는 `Array`이다.

```kotlin
class Array<T>(val size: Int){
  operator fun get(index: Int): T{...}
  operator fun set(index: Int, value: T) {...}
}
```

위와 같은 경우는 Read, Write 모두를 수행해야 함으로 Covariant, Contravariant 모두가 될 수 없다. 만약에 이를 copy하는 함수를 만들었다고 가정했을 때, 다음과 같은 오류가 발생할 것이다.

```kotlin
fun copy(from: Array<Any>, to: Array<Any>) {
    assert(from.size == to.size)
    for (i in from.indices)
        to[i] = from[i]
}

//...

val ints: Array<Int> = arrayOf(1, 2, 3)
val any = Array<Any>(3) { "" }
copy(ints, any) // !!! ERROR
//   ^ type is Array<Int> but Array<Any> was expected
```

오류가 나온 이유를 이제 감을 잡을 것이다. Generic에서의 subtype의 관계는 사라지기 때문에 (즉, invariant이 되기 때문에) `Array<Int>`와 `Array<Any>`는 누가 누구의 subtype이 아닌, 아무런 관련이 없는 관계가 된다.

자, 그럼 `copy()`를 `out`을 활용하여 스마트하게 바꾸어보자. 우리는 `from`에 오는 Array는 source의 역할로서, to (dest)에 copy가 될 element임을 알 수 있고, 이들은 Read-only로 만들어야 한다는 것을 알 수 있다. 즉, `from`은 Read-Only여야 하고, 이는 covariant로 나타낼 수 있다는 것이다.

```kotlin
fun copy(from: Array<out Any>, to: Array<Any>) {
    assert(from.size == to.size)
    for (i in from.indices)
        to[i] = from[i]
}
```

방금 진행한 것이 바로 **Type Projection**이다. `from`에 variance keyword `out`을 붙여줌으로서 `from`이 단순한 배열이 아닌 제한된 (Restricted == Projected) 배열로서, Read-Only로 사용한다는 것이다.

당연하게도 `in` 역시 사용 가능하다. 이는 공식 Docs에 있는 예제인 `fill()`의 구현에서 볼 수 있다. 여기서 보면 `dest`는 위 `copy()`의 `to`의 위치로, `value`로 채워지는 대상 Array이며, 오직 write만 일어나니 contravariance를 적용할 수 있다.

```kotlin
fun fill(dest: Array<in String>, value: String) {
    for(i in dest.indices)
      dest[i] = value
 }
```

### Star Projection

Type argumet에 _무엇이 올지 모르는데_, 그래도 안전한 Generic을 사용하고 싶을 수도 있다. 다음과 같은 경우에는 **Star Projection**이라는 것을 사용할 수 있다.

> 💫 Star Projection은 `Foo<*>`처럼 사용되는 Projection이며, 한번 type이 정해지면 해당 type만 받을 수 있다.

즉, `Foo<*>`는 해당 type parameter에 어떤 type이 들어올지 몰라도 안전하게 사용할 수 있으며, `Any` type과는 달리, 한번 결정이 되면 해당 type만 받을 수 있다.

> 💫 Star Projection은 Type이 정해지기 전까지는 `Any?`로 취급이 된다.

Star Projection은 Variance에 대해서 다음과 같은 특징을 가진다.

1. `Foo<*>`를 `Foo<out T: TUpper>` 와 같이 사용했을 경우, star-projection은 이를 `Foo <out TUpper>`로 이해한다. 즉, `T`에 _무엇이 올지는 모르지만_ 적어도 안전하게 이 value를 `TUpper`로 Up-Casting하여 읽어드릴 수 있다는 것이다.
2. `Foo<*>`를 `Foo<in T>`와 같이 사용했을 경우, star-projection은 이를 `Foo<in Nothing>`와 같은 로 의미한다. 즉, `T`가 unknown 상태일 경우 안전하게 write operation을 진행할 수 없다는 것이다.
3. `Foo<*>`를 `Foo<T: TUpper>`와 같이 사용했을 경우, star-projection은 이를 read operation에 대해서는 `Foo<out TUpper>`, write operation에 대해서는 `Foo<in Nothing>`을 의미한다.

Generic Type은 여러 개가 사용될 수 있고, Projection 역시 각각에게 일어난다. 즉, 다음과 같이 star projection이 사용되면 다음과 같은 의미임을 알 수 있다.

> 만약 `interface F<in T, out U>`와 같은 type이 정의되어 있을 경우,
> `F<*, String>` == `F<in Nothing, String>`  
> `F<Int, *>` == `F<Int, out Any?>`  
> `F<*, *>` == `F<in Nothing, out Any?>`

##Generic Functions
당연한 이야기이지만, Generic은 class 뿐만이 아니라 function에도 사용이 된다.

```kotlin
fun <T> singletonList(item: T): List<T> {
    // ...
}

val l = singletonList<Int>(1)

// Context만으로 type inference가 가능할 경우, Type Argument는 생략이 가능하다.
val l = singletonList(1)
```

Extension Function 역시 다음과 같이 Generic을 적용할 수 있다.

```kotlin
fun <T> T.basicToString(): String { // extension function
    // ...
}
```

## Upper Bound and `where`

다음과 같은 Generic이 있다고 가정을 해보자.

```kotlin
fun <T> onlyEven(list: T){
  return list.filter{it % 2 == 0} //!!! Error: Unresolved reference: filter
}
```

위와 같은 코드는 컴파일 에러가 발생한다. Compiler 입장에서는 list에 어떤 type이 올 줄 알지 못하고, 해당 type에 filter가 가능할 지에 대해서는 더더욱 알지 못한다.

여기서 UpperBound가 필요하다. 즉, 올 수 있는 Generic에 제한을 걸어주는 것이다.

```kotlin
fun <T: Comparable<T>> onlyEven(list: T){
  return list.filter{it % 2 == 0} //!!! Error: Unresolved reference: filter
}
```

저 `T: TUpper`와 같은 형식에서의 `TUpper`는 Upper bound이다. 즉, `T`의 위치에는 `T`, 혹은 `T`를 상속하는 subtype/subclass가 와야 한다.

```kotlin
fun <T : Comparable<T>> sort(list: List<T>) {  ... }

sort(listOf(1, 2, 3)) // OK. Int is a subtype of Comparable<Int>
sort(listOf(HashMap<Int, String>())) // Error: HashMap<Int, String> is not a subtype of Comparable<HashMap<Int, String>>
```

Upper Bound가 존재하지 않을 때 Default Upper Bound는 `Any?`이다.

가끔, 여러 개의 제한이 필요한 경우가 있다. 여러 개의 제한 모두를 Implement했을 경우에만 허용을 하도록 하는 제한이 필요할 수도 있다. 이런 경우에 필요한 keyword가 바로 `where`이다.

> 💫 `where`는 Type Parameter에 대한 두 개 이상의 Upper Bound가 필요할 때 사용된다. where의 조건을 모두 만족한 type만 허용을 시켜준다.

```kotlin
fun <T> copyWhenGreater(list: List<T>, threshold: T): List<String>
    where T : CharSequence,
          T : Comparable<T> {
    return list.filter { it > threshold }.map { it.toString() }
}
```

위의 예시에서 `T`는 `CharSequence` 와 `Comparable`모두를 implement한 type만이 올 수 있다.

## Type Erasure

Generic은 compile time에 type을 정하고 컴파일을 진행한다. 하지만 run time에는 generic type은 이러한 정보를 모두 지운다. `Foo<Bar>`, `Foo<Baz?` => `Foo<*>`와 같은 상태가 된다.
이와 같은 이유로 runtime에 type을 확인하는 `is`연산자는 제한된다.

```kotlin
ints is List<Int> //(X)
list is T //(X)
```

다만 Star Projection으로 바꾸는 Type Erasure의 특성 때문에 다음과 같은 코드는 컴파일이 된다.

```kotlin
if (something is List<*>) {
    something.forEach { println(it) } // The items are typed as `Any?`
}
```

또한, 다음과 같이 Generic임에도 불구하고, 실제 Generic을 사용하지 않고 type을 check하는 경우 또한 컴파일이 된다.

```kotlin
fun handleStrings(list: MutableList<String>) {
    if (list is ArrayList) {// Check without type argument
        // `list` is smart-cast to `ArrayList<String>`
    }
}
```

## Underscore operator

최근에 생긴 `_` operator 또한 type argument에 에서 사용할 수 있다. 2개 이상의 type argument가 필요한 Generic중 하나를 명시적으로 넣고, 다른 하나에 `_`를 넣으면 `_`에 대해서는 type inferrence가 진행된다!

###### Reference

[[Java] Generic에 대한 관찰 -2](https://velog.io/@kasania/Java-Generic%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B4%80%EC%B0%B0-2)
[코틀린 제네릭, in? out?](https://medium.com/mj-studio/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%A0%9C%EB%84%A4%EB%A6%AD-in-out-3b809869610e)
[[Java] Java 제네릭의 형 변환(covariant & contravariant)](https://sabarada.tistory.com/124)
[Generic - Invariance, Covariance, Contravariance](https://www.myanglog.com/Generic%20-%20%20Invariance,%20Covariance,%20Contravariance)
[[Kotlin] 한 방에 정리하는 코틀린 제네릭(kotlin generic) - in, out, where, reified](https://readystory.tistory.com/201)
