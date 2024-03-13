# 범위 지정 함수 (Scope Function)
> 대상 객체에 대해 범위 지정 함수를 호출하면 임시 block 이 형성되는데, 이 scope 내에서 객체에 대한 작업을 정의할 수 있습니다.

## let
```kotlin
public inline fun <T, R> T.let(block: (T) -> R): R
```
- 파라미터로 명시적 전달
- block 의 마지막 줄을 return 한다.
- ex) 객체를 생성하거나 사용하는 시점에서 이름을 부여하여 다양한 작업을 수행시키고 결과를 돌려받고 싶을 때 사용한다. (혹은 null check)
  ```kotlin
  person.name?.let { personName ->
    "my name is $personName"
  }
  ```

## also
```kotlin
public inline fun <T> T.also(block: (T) -> Unit): T
```
- 파라미터로 명시적 전달
- 수신 객체 자체를 return 한다.
- ex) 객체에게 명령을 내리기 직전 추가적인 작업을 함께 수행시키고 싶을 때 사용한다.
  ```kotlin
  // apply 와 다르게 인자를 넘겨 이름을 정할 수 있다.
  persons.first()
    .also { firstPerson ->
      printIn("이름 : ${firstPerson.name}")
    }
  ```

## apply
```kotlin
public inline fun <T> T.apply(block: T.() -> Unit): T
```
- receiver 로 암시적 전달
- 수신 객체 자체를 return 한다.
- ex) 객체에 무언가를 적용할 때 사용한다. (객체 생성 시점에서 초기화 할 때 유용)
  ```kotlin
  Person().apply {
    age = 30,
    name = "memeze"
  }
  ```

## run
```kotlin
public inline fun <T, R> T.run(block: T.() -> R): R

public inline fun <R> run(block: () -> R): R
```
- receiver 로 암시적 전달
- 수신 객체 없이도 사용 가능하지만 block 내에 객체를 명시해야 한다.
- block 의 마지막 줄을 return 한다.
- `run`과 유사하지만 객체를 생성해서 할당하기전에 사용이 가능하다는 차이가 있다.
- ex) 특정 동작 수행 후 결과값 리턴 시 사용한다.
  ```kotlin
  val nameLength = person.run { name.length }
  ```

## with
```kotlin
public inline fun <T, R> with(receiver: T, block: T.() -> R): R
```
- receiver 로 암시적 전달
- 다른 scope function 들과 다르게 extension function 이 아니라 객체를 argument 로 받아 사용한다.
- block 의 마지막 줄을 return 한다.
- ex) 이미 생성된 객체에 일괄적인 작업을 처리할 때 사용한다.
  ```kotlin
  // with 사용하기 전 코드
  person.age = 30
  person.name = "memeze"

  // with 사용
  with(person) {
    this.age = 30
    this.name = "memeze"
  }
  ```
