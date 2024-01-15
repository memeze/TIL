# 범위 지정 함수 (Scope Function)
> 대상 객체에 대해 범위 지정 함수를 호출하면 임시 block 이 형성되는데, 이 scope 내에서 객체에 대한 작업을 정의할 수 있습니다.

## let
```kotlin
public inline fun <T, R> T.let(block: (T) -> R): R
```
- 파라미터로 명시적 전달
- block 마지막 줄 return
- ex) null check

## also
```kotlin
public inline fun <T> T.also(block: (T) -> Unit): T
```
- 파라미터로 명시적 전달
- 수신 객체 자체 return
- ex) 프로퍼티를 바꾸지 않고 동작을 수행할 때 사용

## apply
```kotlin
public inline fun <T> T.apply(block: T.() -> Unit): T
```
- receiver 로 암시적 전달
- 수신 객체 자체 return
- ex) 객체 선언과 동시에 초기화 시 사용

## run
```kotlin
public inline fun <T, R> T.run(block: T.() -> R): R

public inline fun <R> run(block: () -> R): R
```
- receiver 로 암시적 전달
- block 마지막 줄 return
- 수신 객체 없이도 사용 가능하지만 block 내에 객체를 명시해야 함
- ex) 특정 동작 수행 후 결과값 리턴 시 사용

## with
```kotlin
public inline fun <T, R> with(receiver: T, block: T.() -> R): R
```
- receiver 로 암시적 전달
- block 마지막 줄 return
- `run`과 비슷하지만 객체를 파라미터로 받아서 사용
