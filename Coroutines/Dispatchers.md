# 📚 디스패처 (Dispatchers)

코루틴은 디스패처를 사용해 스레드(또는 스레드풀)을 결정할 수 있다.

</br>

## Default Dispatcher
- 디스패처를 설정하지 않으면 기본적으로 설정되는 디스패처
- CPU 집약적인 연산을 수행하도록 설계된 디스패처
- CPU 개수와 동일한 수의 스레드 풀을 가진다. (최소 두개 이상)
- `runBlocking`은 디스패처가 설정되어 있지 않으면 자신만의 디스패처를 사용하여 Dispatchers.Default 가 자동으로 선택되지 않는다.

</br></br>

## Main Dispatcher
- 안드로이드에서 메인 스레드는 UI와 상호작용하는데 사용하는 유일한 스레드
- 안드로이드에서는 기본 디스패처로 메인 디스패처를 주로 사용한다. (복잡한 연산을 한다면 Default 사용)

</br></br>

## IO Dispatcher
- 파일을 읽고 쓰거나 블로킹 함수를 호출하는 경우 같이 I/O 연산으로 스레드를 블로킹할 때 사용한다.
- Dispatchers.IO 는 64개(또는 더 많은 코어가 있다면 해당 코어의 수)로 제한된다.
- Dispatchers.Default 와 Dispatchers.IO 는 같은 스레드풀을 공유한다.

</br></br>

## 디스패처 제한
- 아래와 같이 limitedParallelism 을 사용하면 같은 기산에 특정 수 이상의 스레드를 사용하지 못하도록 제한할 수 있다.
- limitedParallelism 은 kotlinx.coroutines 1.6 버전에서 도입
  ```kotlin
  private val dispatcher = Dispatchers.Default
      .limitedParallelism(5)
  ```
- Dispatchers.IO 에는 `limitedParallelism` 함수를 위해 정의된 특별 작동 방식이 있다.  
  limitedParallelism 함수는 독립적인 스레드 풀을 가진 새로운 디스패처를 만든다. (스레드 수가 64개로 제한되지 않음)

</br></br>

## 정해진 수의 스레드풀을 가진 디스패처
- Excutors 클래스를 스레드의 수가 정해져 있는 스레드 풀이나 캐싱된 스레드 풀을 만들 수 있다.
- `asCoroutineDispatcher` 함수를 사용하여 디스패처로 변형하는 것도 가능하다.
  ```kotlin
  val NUMBER_OF_THREADS = 20
  val dispatcher = Executors
      .newFixedThreadPool(NUMBER_OF_THREADS)
      .asCoroutineDispatcher()
  ```
- `asCoroutineDispatcher` 로 만들어진 디스패처는 `close` 함수로 닫아야 한다. (그렇지 않으면 메모리 누수가 생길 수 있음)
- 또 다른 문제는 정해진 수의 스레드 풀을 만들면 효율적으로 사용하지 않는다.  
  (사용하지 않는 스레드가 다른 서비스와 공유되지 않고 살아있는 상태로 유지됨)

</br></br>

## 싱글스레드로 제한된 디스패처
- 다수의 스레드를 사용하면 공유 상태로 인한 문제를 생각해야 한다.
- 싱글스레드를 사용하면 동기화를 위한 조치가 필요하지 않다.
- 아래와 같은 방법으로 싱글스레드로 제한된 디스패처를 생성한다.
    1. Executors 를 사용하여 싱글스레드 디스패처를 생성한다.
       ```kotlin
       val dispatcher = Executors.newSingleThreadExecutor().asCoroutineDispatcher()
       ```
       - 디스패처가 스레드 하나를 액티브한 상태로 유지하고 있으며, 더 이상 사용되지 않을 때는 스레드를 반드시 닫아야 하는 문제점이 있다.
  
    2. Dispatchers.Default 나 병렬처리를 1로 제한한 Dispatchers.IO 를 주로 사용한다.
       ```kotlin
       val dispatcher = Dispatchers.Default.limitedParallelism(1)
       ```
       - 단 하나의 스레드만 가지고 있기 때문에 이 스레드가 블로킹되면 작업이 순차적으로 처리되는 것이 가장 큰 단점이다.

</br></br>

## Dispatchers.Unconfined (제한받지 않는 디스패처)
- 이 디스패처는 스레드를 바꾸지 않는다.
- 디스패처가 실행되면 시작한 스레드에서 실행된다.
- 디스패처가 재개되었을 때는 재개한 스레드에서 실행된다.
```kotlin
suspend fun main(): Unit =
    withContext(newSingleThreadContext("Thread1")) {
        var continuation: Continuation<Unit>? = null

        launch(newSingleThreadContext("Thread2")) {
            delay(1000)
            continuation?.resume(Unit)
        }

        launch(Dispatchers.Unconfined) {
            print(Thread.currentThread().name)  // Thread1

            suspendCancellableCoroutine<Unit> {
                continuation = it
            }

            print(Thread.currentThread().name)  // Thread2

            delay(1000)

            print(Thread.currentThread().name)  // kotlinx.coroutines.DefaultExecutor (delay가 사용한 스레드)
        }
    }
```
- 이 디스패처는 단위 테스트할 때 유용하다.
- Unconfined 를 사용하면 모든 작업이 같은 스레드에서 실행되기 때문에 연산의 순서를 쉽게 통제할 수 있다.
- 하지만, **kotlinx-coroutines-test** 의 `runTest` 를 사용하면 이 방법은 필요하지 않다.
- 성능적인 측면에서 보면 스레드 스위칭을 일으키지 않는다는 점에서 Unconfined 디스패처 비용이 가장 저렴하다.

</br></br>

## Dispatchers.Main.immediate
- 코루틴을 배정하는 것도 비용이 든다.
- withContext 가 호출되면 코루틴은 중단되고 큐에서 기다리다가 재개된다.
- 이미 Dispatchers.Main 에서 `withContext(Dispatchers.Main)` 이 호출될 경우 쓸데없는 비용이 발생한다.
- 메인 스레드를 기다리는 큐가 쌓여있을 경우 withContext 때문에 사용자 데이터는 약간의 지연이 발생한다.
- 반드시 필요한 경우에만 배정하기 위해 `Dispatchers.Main.immediate` 를 사용한다.
- 다른 Dispatcher 에서 호출했을 경우는 Dispatchers.Main 과 동일하게 동작한다.

</br></br>

## 컨티뉴에이션 인터셉터
- 디스패칭은 코틀린 언어에서 지원하는 컨티뉴에이션 인터셉션을 기반으로 작동한다.
- `ContinuationInterceptor` 라는 코루틴 컨텍스트는 코루틴이 중단되었을 때 interceptContinuation 메서드로 컨티뉴에이션 객체를 수정하고 포장한다.
- releaseInterceptedContinuation 메서드는 컨티뉴에이션이 종료되었을 때 호출된다.
```kotlin
public interface ContinuationInterceptor : CoroutineContext.Element {
    companion object Key : CoroutineContext.Key<ContinuationInterceptor>

    fun <T> interceptContinuation(continuation: Continuation<T>): Continuation<T>

    fun releaseInterceptedContinuation(continuation: Continuation<*>) {}

    ...
}
```

</br></br>

## 작업 종류에 따른 각 디스패처 성능 비교
||중단|블로킹|CPU 집약적인 연산|메모리 집약적인 연산|
|:---:|:---:|:---:|:---:|:---:|
|싱글스레드|1,002|100,003|39,103|94,358|
|Dispatchers.Default (스레드 8개)|1,002|13,003|8,473|21,461|
|Dispatchers.IO (스레드 64개)|1,002|2,003|9,893|20,776|
|스레드 100개|1,002|1,003|16,379|21,004|

- 단지 중단할 경우에는 사용하고 있는 스레드 수가 얼마나 많은지는 문제가 되지 않는다.
- 블로킹할 경우에는 스레드 수가 많을수록 모든 코루틴이 종료되는 시간이 빨라진다.
- CPU 집약 연산은 `Dispatchers.Default` 가 가장 좋다.
- 메모리 집약 연산을 처리하고 있다면 더 많은 스레드를 사용하는 것이 좀 더 낫다. (차이는 별로 없음)
