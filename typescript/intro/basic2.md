# 내가 원하는대로 데이터를 관리할 수 있다니 이거 완전 럭키비키자나 ✨✨ - 타입스크립트 기본 문법 (Typescript Basics 2)
- 보통 [전 단계](https://github.com/KyumKyum/DEV_SALAD/blob/main/typescript/intro/basic1.md)이후, 일반적인 순서는 조건문과 반복문이다. 
- 하지만 Typescript에서 타입 시스템을 조금 더 다뤄보고자 한다. 이 과정이 이후 타입스크립트 개발에 있어서 필수적이라는 것을 알게 될 것이다. :)
---
### 0. 배울 것 (Contents)
- 상수 (Constants)
- 타입과 인터페이스 (Type & Interface)

> Basics에 있는 개념들은 앞으로 정말 많이 사용될 개념이니 꼭꼭 익히도록 하자!
---
### 1. 상수 (Constants)
> ✏️ 한번 다음 코드의 결과를 예상해보자.
```typescript
let name = "Jay Lim"

name = "Kyumerican0" 

console.log(name) // Kyumerican0
```
- 예상했듯이, `name`에 새로운 값이 덮어 씌워지면서, 'Jay Lim' 대신 'Kyumerican0'가 출력되는 것을 볼 수 있다.
- 개발을 할 때, 우리는 초기 값이 변하게 만들고 싶지 않은 경우가 많다. 우리 이름에 대한 변수를 선언했는데, 그 값이 변해서 우리 이름이 아니라 다른 사람이 이름이 출력되면 어이 없을 것이다.

- 이를 해결하기 위한 2가지 방법이 있다.
1. 해당 변수를 조심해서 관리해서, 새로운 값이 할당 안되게 만든다. 
    - 일단, 난 나 자신을 믿지 못한다...ㅎ ~~개발할 때 가장 큰 적은 나 자신~~ 우리가 아무리 조심한다고 해도, 사람은 늘 실수를 하는 존재이며, 정신없이 개발을 하다보면 나도 모르는 사이에 새로운 값이 들어가는 경우가 있다.
> 💡 이렇게 기계가 아닌, 사람 본연의 실수로 일어나는 에러를 '휴먼 에러 (Human Error)'라고 한다.

2. **상수 (Constant)** 사용하기!
- 다음 예시를 보자.
```typescript
const name = "Jay Lim"

name = "Kyumerican0" // X, Error: 여기서 에러가 난다.
```

- **-NF-적 설명**  
    - 상자에 물건을 넣은 후, 얼음 빔을 쏜다고 생각해보자. 상자에 들어간 물건은 이제 꽝꽝 얼어버려 다른 물건을 넣으려고 해도 넣을 수 없게 될 것이다. 즉, 변수에 데이터를 넣은 후 얼음빔을 쏴 얼려버려, 다른 값이 들어오는 것을 원천적으로 막아버리는 것이 상수 선언이다.
- **-ST-적 설명**
    - 상수 선언을 하면, 시스템은 해당 값을 불변(immutable)하는 값으로 인식을 한다. 즉, 상수 선언을 한다면 해당 변수는 변하지 않는 값이라고 선언을 한 것이고, 시스템 상으로 해당 변수 새로운 값이 들어오는 것을 원천적으로 막아 불변성을 지킨다. 이런 특수한 변수를 '상수'라고 우리는 이야기한다.

> 💡 생각보다 휴먼 에러는 자주 일어난다! 해당 값이 변하지 않는 값이라는 확신이 든다면, 상수 선언을 하는 습관을 들이자!
---
### 2. 타입과 인터페이스 (Type & Interface)