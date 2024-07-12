# 어머! 우리 아이가 걸음마를 땠어요! 🎉🎉 - 타입스크립트 기본 문법 1 (Typescript Basics 1)
- 일단 축하의 빵빠레!! 🎉🎉 이제 힘든 세팅은 다 끝났다!!
- 이제 편안한 마음으로, 쉬운 문법부터 조금씩 배워보자 :)
---
### 0. 배울 것 (Contents)
- 변수 (Variables)
- 객체 (Constants)
- 배열 (Arrays)
- 함수 (Functions)

> Basics에 있는 개념들은 앞으로 정말 많이 사용될 개념이니 꼭꼭 익히도록 하자!
---
### 1. 변수 (Variables)
- 모든 프로그래밍의 가장 기본이 되는 변수 선언. 변수는 테이터를 저장하는 공간이다. 
```typescript
let value = 1;
```

- `value`라는 이름을 부여한 공간에 `1`이라는 숫자를 저장하였고, 공간의 이름을 통해 해당 데이터에 접근할 수 있다.

```typescript
console.log(value)  // 1
```

- 'Type'script라는 이름처럼, 데이터를 저장할 공간에 저장할 데이터의 '종류(type)'에 대해서 이야기해줄 수 있다. 그렇게 된다면, 그 공간에는 그 종류의 데이터만이 저장이 될 수 있다.
```typescript
let value:number = 1; // value라는 공간에는 숫자형 데이터만이 올 것이고, 이 공간에 1을 저장했다.
let str:string = 'Hello'; // str라는 공간에는 문자열 데이터만이 올 것이고, 이 공간에는 'Hello'를 저장했다.

let str:string = 1; // X, 해당 케이스는 에러가 난다. 문자열 데이터가 올 곳에 숫자형 데이터가 왔기 때문이다.
```

> 💡 변수는 과일 (데이터)를 담는 일종의 '상자' 개념이다. 이 상자에 사과만 담는다고 적었으면, 그 상자에는 사과만이 담길 수 있는 것이다.  

> 🌟 용어 통일!
- 위와 같이 데이터를 저장할 공간을 선언하는 것을 `변수 선언`이라고 이야기를 하고, 해당 변수에 들어올 수 있는 종류를 정의해서 제한하는 것은 `타입 정의`라고 이야기를 한다. 
- 타입 정의는 변수를 선언한 후, `:` 뒤에다가 정의한다.

---
### 2. 객체 (Objects)
- 위에서 배운 개념으로 한번 '나 자신'을 이름, 나이, 학과로 정의해보자.
    - 예를 들어, 나 자신을 다음 기준으로 정의한다면,
        - 이름: Jay Lim
        - 나이: 24 ~~25살이지만 만 나이로 할꺼에요~~
        - 학과: Information System & Computer Science
- 변수는 다음과 같이 정의할 수 있다.

```typescript
let name:string = 'Jay Lim'
let age:number = 24
let major:string = "IS & CS"
```
- 하지만, 각 변수는 이름, 나이, 학과만을 뜻하지, 이것이 '나 자신'이라는 하나의 거대한 정보를 이루는 데이터라는 정보는 없다. 
- 우리는, 이러한 정보를 '나 자신'이라는 **하나의 데이터**로 묶을 필요성이 있는데, 여기서 사용되는 개념이 바로 **객체 (Object)** 이다.

```typescript
let myself = {
    name: 'Jay Lim',
    age: 24,
    major: "IS &   CS"
}
```
- 정의되는 방법이 조금 바뀌었다. **타입**을 정의하는 것이 사라지고, `:` 뒤에 바로 내가 넣고자 하는 데이터를 바로 정의한다.
- `myself`라는 객체 안에, 나를 정의하는 데이터 3개를 넣은 형태이다. 이 객체는 다음과 같은 방법으로 사용 가능하다.

```typescript
console.log(myself.name); //Jay Lim
console.log(myself.age); //24
console.log(myself.major); // IS & CS
```
> 🌟 용어 통일!
- 위 `myself`와 같이, 데이터의 묶음을 (`{...}`) 정의한 것은 `객체 (Object)`라고 정의를 한다.
- 객체를 구성하는 데이터들 (`age: 24`)은 `필드 (Field)`라고 정의를 한다.
- `객체.필드`와 같은 문법으로 각 객체의 필드에 접근할 수 있다.

> 💡 응용: 객체 안의 객체
- 당연히, 객체 안의 필드로 또 다른 객체가 들어올 수 있다! 위에서 정의한 `myself`객체의 `name`필드를 성(`lastName`)과 이름(`firstName`)으로 세분화한다면, 다음과 같이 정의할 수 있을 것이다.

```typescript
let myself = {
    name: {
        firstName: 'Jay',
        lastName: 'Lim'
    },
    age: 24,
    major: "IS & CS"
}
```
- 다음과 같이 접근이 가능할 것이다. :)

```typescript
console.log(myself.name.firstName); // Jay
```

> ❓️ 객체는 변수처럼 타입 지정을 못하나요?
- 당연히 가능하고, 객체의 타입 지정 방식이 위에서 설명한 타입 지정보다 지향되는 방법이다. 하지만, 해당 방법을 공부하기 위해서는 '타입과 인터페이스 (Type and Interface)'에 대해서 알아야하니, 이는 다음 기본 단계에서 설명하는 것으로 한다.
---
### 3. 배열 (Arrays)
- 우리는 객체라는 개념을 통해서 데이터를 풍부하게 정의하는 방법을 배웠다.
- 배열은, **여러 개의** 데이터를 정의하는 방법을 이야기한다.

> 💡 학생 5명, 덤벨 12개, 커피 36잔 등등, 여러개의 나열되어 있는 데이터는 배열로 정의한다!

- 배열은 예시를 보면 곧바로 이해가 가능하다. 학생들의 이름이 나열되어있는 배열을 생각해보자.
```typescript
const nameList: string[] = ['Jay', 'Eve', 'Sarah', 'Ethan']
console.log(nameList[0]) // Jay
console.log(nameList[2]) // Sarah
```
> ✏️ 깜짝 상식! 프로그래밍 언어에서 숫자는 0부터 시작한다! 0,1,2,3.... 이 개념이 익숙해지자!

- 배열에는 당연히 객체도 들어올 수 있다. 이번에는, 학생들의 이름 리스트가 아니라, 학생 정보 리스트이다.
```typescript
const studentList = [
    {
        name: 'Jay',
        age: 24,
        major: "IS & CS"
    },
    {
        name: 'Eve',
        age: 21,
        major: "Literature"
    },
    {
        name: 'Sarah',
        age: 23,
        major: "Mechanical Engineering"
    },
    {
        name: 'Ethan',
        age: 22,
        major: "Law"
    }
]
```

- 심지어, 배열은 꼭 같은 데이터만 들어오라는 법도 없다!
```typescript
const studentList = [
    {
        name: 'Jay',
        age: 24,
        major: "IS & CS"
    },
    {
        name: 'Eve',
    },
    0,
    {
        name: {
            firstName: 'Joshua',
            lastName: 'Park'
        },
        age: 24,
    }
]
```
- 이런 말도 안되는 배열도 충분히 가능한 타입스트립트이다. 이런 엉망진창인 배열에 대해서도 우리는 타입을 지정할 수 있는가? 당연히 가능하다! 하지만 이 부분 부터는 타입과 인터페이스에 대한 내용을 어느정도 알아야 하기도 하고, 충분히 어려운 내용이어서 심화 단계에서 설명하려고 한다.
---
### 4. 함수 (Functions)
> ✏️ 잠깐 눈을 감아보고, '함수'의 개념에 대해서 한번 생각해보자. 함수란 무엇인가?
- 난 함수를 다음과 같이 정의한다: **주어진 입력값에 해당하는 결과값을 만들어내는 로직.**
    - **쉬운 설명:** 
        - 함수는 기계이다. 내가 어떤 것을 이 기계에 넣으면, 이 기계는 그것에 대한 무언가를 내뱉는다. 같은 것을 넣으면 같은 무언가를 내뱉을 것이다.
    - **간단한 설명:** 
        - $f(x) = y$
> 💡 중요한 것은, '같은 입력값에 대해서, 어떠한 절차를 거치고 같은 결과값을 도출한다는 것이다!' 프로그래밍 세계에서는 이 명제는 항상 참이 아니지만, 현재 레벨에서는 이러한 개념으로 함수를 접근하자. :)

- Typescript에서의 함수는 이에 걸맞는 구조를 가지고 있다. 예시를 먼저 들겠다. 2개의 입력값을 받아서 두 값을 더한 값을 결과값으로 반환하는 간단한 더하기 함수를 정의해보겠다.
```typescript
function add(num1: number, num2: number): number {
    return num1 + num2;
}
```
- 다음과 같은 구성으로 구현이 되어있는 것을 확인할 수 있다.
    1. 함수임을 알려주는 `function` 키워드
    2. 함수의 이름인 `add`
    3. 두 개의 입력값 `num1`, `num2`. 그리고 입력값에 타입 정의 (`number`)
    4. 해당 함수의 반환값의 타입 정의 (`number`)
    5. 함수를 실행하면 동작하는`{...}`
    6. 결과값을 반환하는 `return` 키워드.
> 💡 현재는 별다른 구성 없이 곧바로 두 입력값을 더한 후에 반환을 하지만, 함수를 실행할 시 {} 내부에 있는 것을 실행시킨다는 점에서 더욱 풍부한 함수를 정의할 수 있다. 

- 함수를 실행하면, 다음과 같은 결과를 줄 것이다.
```typescript
console.log(add(1,2)); // 3
```
> ✏️ 위 내용을 참고하여, 아래 함수 역시 구성을 분석해보자. 참고로, void 타입은 '아무것도 없는'이라는 뜻임을 참고하자.
```typescript
function introduce(name: string, hobby: string): void {
    console.log(`Hello! My name is ${name}`) // 여기서 '' 가 아닌, ``임을 주목하자! ``은 문자열 안에서 데이터를 사용할 수 있게 해준다.
    console.log(`I do ${hobby} for fun!`)
    console.log('End of my introduction! Now, I want to hear about you.')
    // return (void 반환은 return을 제외할 수 있습니다.)
}
```