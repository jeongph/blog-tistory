[Kotlin] Scope functions(let, run, with, apply, also)가 헷갈리면

# Intro

### TL;DR

| Function | Object reference | Return value   | Is extension function                        |
| -------- | ---------------- | -------------- | -------------------------------------------- |
| `let`    | `it`             | Lambda result  | Yes                                          |
| `run`    | `this`           | Lambda result  | Yes                                          |
| `with`   | `this`           | Lambda result  | No: takes the context object as an argument. |
| `apply`  | `this`           | Context object | Yes                                          |
| `also`   | `it`             | Context object | Yes                                          |

- `null`이 아닌 객체를 지역에서 사용하고 싶을 때 = `let`
  - e.g. `if(object != null){ .. }`
- 객체의 생성과 동시에 값(configuration)을 지정 할 때 = `apply`
- 객체의 값을 지정하고 바로 특정 기능을 실행할 때 = `run`
- 현재의 객체로 추가적인 동작을 수행할 때 = `also`
- 객체의 지역 범위를 지정하여 재사용 할 때 = `with`

# Contents

- 기본적으로 모든 scope function들은 지역 변수의 범위 제한을 위해 사용 할 수 있다.
- 블럭 매개변수로 람다 리시버를 받으면 this를(`run`, `with`, `apply`), 람다 인자를 그대로 받으면 it을 사용(`let`, `also`)한다.

### `let`
``` kotlin
// 구현체
public inline fun <T, R> T.let(block: (T) -> R): R {
    return block(this)
}
```
- 제너릭 `T`의 확장함수 형태로, 매개변수로 자기자신(`T`)를 사용하여 람다(`block`)을 실행하고, 람다의 리턴값 `R`을 리턴한다.
- 컴파일 할 때 매개변수의 `Nullable`을 체크하기 때문에, `if( object != null )`를 생략할 때 사용할 수 있다.
- __특성상 null 체크와 동시에 지역에서 사용하거나, 수신 객체를 활용하여 새로운 데이터(리턴값)을 만들때 사용한다.__
- e.g. )
``` kotlin
// 객체의 내부 프로퍼티를 리턴하거나 없는 경우 초기값을 리턴
val pageUrl = page?.let{ it.url } ?: "default url"

// 특정 객체에서 가져온 값을 조합하여 새로운 값으로 리턴
val personFullName = person.let {
  it.firstName = "Jeong-Uk"
  it.lastName = "Park"

  "$firstName $lastName" // R
}
```

### `run`
``` kotlin
// 구현체
public inline fun <T, R> T.run(block: T.() -> R): R {
    return block()
}
```
- 제너릭 `T`의 확장함수 형태로, 매개변수로 

# References
1. [Scope functions](https://kotlinlang.org/docs/scope-functions.html)