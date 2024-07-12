# Kotlin

## Kotlin을 배우고자 결심한 이유에는 실로 놀라운 여러가지 이유가 있다. 하지만 여백이 부족하여 이를 적지 않겠다. 😎😎

...라고는 이야기를 하지만, Kotlin을 배우기로 한 이유는 다음과 같다.

- Null Safety:

  - 정말이지 JS 개발자로서 너무 힘들었던 것이 시도때도 발생하는 원인 모를 null 값들이었다. GraphQL로 type을 어느정도 강제했음에도 null은 끊임없이 나오고, 이로 인해 죽는 서버와 시스템은 계속 나를 괴롭혔다. Kotlin은 기본적으로 Nullable을 지원해서 JS와 같이 Null이 되므로서 발생하는 오류에 대해서도 안전하지만, null일 경우 처리할 수 있는 다양한 mechanism을 제공하기 때문에 null이 발생하면 안되는 곳에서는 절대로 발생하지 않도록 강제할 수 있는 강력한 시스템이 여러개 존재하여, 이를 유연하게 사용할 수 있다.

- Concise

  - 난 아직도 Java의 exception 처리와 class들의 난무로 인한 악몽을 잊지 못한다. Kotlin으로 개발된 서비스의 코드 일부를 읽어볼 기회가 있었는데, 첫 인상은 매우 "깔끔하다"였다. Lamda, getter & setter 등등 여러 필수적인 코드를 매우 간결하게 표현하여 readble하고 짧게 표현할 수 있다.

- Verified

  - Kotlin을 공부하다보면 여러 다른 언어의 익숙한 문법이나 기능들이 보일 것이다. 이미 다른 프로그래밍 언어에서 검증된 기능과 기법들을 모두 가지고 온 언어이기 때문이다. ?. 같은 null safety operator 봤을 때 진짜 반갑긴 하더라.

- 이외에도 Functional Programming을 인한 readability와 maintainability의 용이함, Type Inference 등등이 있다.

여기에는 내가 공부한 Kotlin의 내용들을 계속 적을 것이다. 기본 문법들에 대해서도 적을 생각이다. 혹시라도 잘못되거나 추가 의견이 있으면 이에 대한 Issue와 PR은 언제나 환영이다!

### Contents

[가장 기본적인 것부터 🔥🔥 - OOP](https://github.com/KyumKyum/DEV_SALAD/tree/main/kotlin/OOP)

// TODO

- Type Related System
