# Type-related system

## is - Type Checking

- is는 값이 원하는 type인지 check 한다.

```kotlin
fun getStringLength(obj: Any): Int? {
    if (obj is String) { // Check if obj is String type
        // if it is, `obj` will be automatically cast to `String`
        return obj.length
    }

    // `obj` is still of type `Any` outside of the type-checked branch
    return null
}
```

## as - Type Casting

- as는 값을 원하는 type으로 casting한다.

```kotlin
a as String
//* Cast variable 'a' as String type
//* ClassCastException if casting is not possible

a as? String
//* Cast variable 'a' as String type
//* null if casting is not possible
```