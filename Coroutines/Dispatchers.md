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

