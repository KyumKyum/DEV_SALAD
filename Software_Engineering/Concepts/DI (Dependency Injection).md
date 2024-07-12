# Dependency Injection

DI는 Kotlin 뿐만이 아니라 Java에서도 매우 중요한 개념이다. 이후에 다룰 @Autowired Annotation에서도 중요한 개념이니, 간단하게 짚고 넘어가겠다.

## Dependnecy

- A가 B를 의존하는 관계하고 가정을 해보겠다. 그러면 다음과 같이 정의될 수 있다.
  > 의존 대상인 B의 속성이 변하거나 추가되면, 그것이 A에게도 영향을 미친다.
- 예를 들어, 운동을 하려고 하는 헬린이가 오늘 할 수 있는 운동은 그 사람이 알고 있는 운동의 종류에 의존한다. 라고도 이야기할 수 있다.

```kotlin
class Newbie {
  val workout: Workout = Workout()

  fun doWorkout {
    workout.grind()
  }
}
```

- Workout이라는 의존 관계를 interface로 추상화하면, 더욱 다양한 운동 종류에 의존할 수 있는 코드를 만들어낼 수 있다.

```kotlin
interface Workout {
  val lightweight: String

  fun grind(){
    print(lightweight)
  }
}

class Squat: Workout {
  override val lightweight: String = "Light Weight Baby!"
}

class BenchPress: Workout {
  override val lightweight: String = "Yeah Buddy!"
}

class Deadlift: Workout {
  override val lightweight: String = "Hoooooooo!"
}

class Newbie {
  val workout: Workout = Squat()
  // val workout: Workout = BenchPress()
  // val workout: Workout = Deadlift()

  fun doWorkout {
    workout.grind()
  }
}

```

- 즉, 의존관계라고 하는 것은,
  1. 의존하는 대상의 성질이 변하면 의존을 하는 대상 또한 영향을 미치며,
  2. Interface로 의존관계를 추상화하면 더욱 많은 의존 관계를 맺음과 동시에 클래스간 관계를 느슨하게 만들 수 있다. (확장성을 위해 고려햐야 하는 것.)

## Dependency Injection

- 그런데, 위 코드에서는 문제가 하나 있다. 바로 어떠한 의존 관계를 가질지에 대한 정보가 내부적으로 직접 정하는 것이다. (주석 처리된 부분 주목). 요청에 따라서 의존관계를 바꿔야 하는 상황에서는 이 방법을 사용할 수 없다.
- 의존관계를 외부에서 정할 수 있다면 해당 문제는 해결이 된다. 뭐, 예를 들어 트레이너가 해당 헬린이에게 오늘 해야할 운동을 알려주면, 그거에 맞춰서 운동을 하는 것 처럼.
- 이처럼 이러한 의존 관계를 외부에서 결정하고 주입하는 것이 DI, 의존관계 주입이다.
- 의존 관계 주입은 다음과 같이 설명할 수 있다. (Tody's Spring 참고)

> 1.  클래스 모델, 코드에는 런타임 시점의 의존관계가 드러나있지 않는다.
>     즉, 인터페이스에만 의존을 하고 있다.
> 2.  이러한 의존 관계는 외부의 제 3의 존재 (컨테이너나 팩토리 등)가 결정한다.
> 3.  외부에서 제공되는 의존 관계에 사용될 객체가 주입됨으로 인해 의존관계는 만들어진다.

- 즉, 런타임 시점에 의존관계를 주입하여 의존관계를 완성한다.

```kotlin
// Method 1: Using constructor
class Newbie(val workout: Workout) {

  fun doWorkout {
    workout.grind()
  }
}

class Trainer {
  val newbie: Newbie = Newbie(Squat())

  newbie.doWorkout()
  //"Light Weight Baby!"

}
```

```kotlin
// Method 2: Using setter
class Newbie {

  var workout:Workout = Squat()
    set(workout: Workout) { // Set is basically gived for 'var'
      field = workout
    }


  fun doWorkout {
    workout.grind()
  }
}

class Trainer {
  val newbie: Newbie = Newbie()
  newbie.worktout = BenchPress()

  newbie.doWorkout()
  //"Yeah Buddy!"
}
```

### Benefits

- DI를 적극적으로 활용하면 다음과 같은 장점이 생긴다.

1. 의존성이 줄어든다. 주입받는 대상이 변하더라도, 구현 자체를 바꿀 필요가 없다.
2. 재사용성이 높아진다.
3. 테스트가 용이해진다. 의존을 주입하는 대상을 주입 받는 대상과 분리하여 테스트가 가능하다.
4. 가독성이 높아진다.
