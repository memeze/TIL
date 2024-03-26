# 동시성 문제
- 여러 스레드에서 하나의 값에 접근하게 되면 동시성 문제가 일어날 수 있다.
- 문제 해결을 위해 아래와 같은 방식을 소개한다.

</br>

## 동기화 블로킹
- 자바에서는 `synchronized` 블록이나 동기화된 컬렉션을 사용하여 동시성 문제를 해결할 수 있다.
- 하지만 `synchronized` 블록 내부에서는 suspend 함수를 사용할 수 없다. (코루틴을 사용할 때는 다른 방식을 사용)

</br></br>

## 원자성
- Atomic 컬렉션들을 사용하여 원자성 연산으로 스레드 안전을 보장한다.
- 하나의 연산에서는 원자성을 가지고 있지만 아래와 같이 전체 연산에서는 원자성이 보장되지 않는다.
  ```kotlin
  private var counter = AtomicInteger()

  fun main() = runBlocking {
      // 1,000,000 번 연산하는 함수
      massiveRun {  
          counter.set(counter.get() + 1)
      }
      print(counter.get())  // ~430467
  }
  ```

</br></br>

## 싱글스레드로 제한된 디스패처
- 하나의 스레드로 제한하는 디스패처를 만들어 해결할 수 있다.
  ```kotlin
  val dispatcher = Dispatchers.IO.limitedParallelism(1)

  fun main() = runBlocking {
      // 1,000,000 번 연산하는 함수
      massiveRun {
          withContext(dispatcher) {
              counter++
          }
      }
      print(counter.get())  // 1000000
  }
  ```
- `코스 그레인드 스레드 한정`
  - 함수 전체를 withContext 로 래핑하여 처리한다.
  - 블로킹 또는 CPU 집약적인 함수 호출시 실행이 느려지게 된다.
    
- `파인 그레인드 스레드 한정`
  - 상태를 변경하는 구문들만 withContext 로 래핑한다.
  - 번거롭지만 코스 그레인드보다 더 나은 성능을 제공한다.

</br></br>

## 뮤텍스
- 첫 번째 코루틴이 lock 을 호출하면 중단 없이 작업을 수행한다.
- 또 다른 코루틴이 lock 을 호출하면 첫 번째 코루틴이 unlock 을 호출할 때까지 중단된다.
- 마찬가지로 다른 코루틴들이 실행될 때마다 작업이 큐에 등록된다.
- 단 하나의 코루틴만이 lock 과 unlock 사이에 있을 수 있다.
- lock 과 unlock 을 직접 사용하지 않고 `withLock` 을 사용할 수 있다. (사용 방식은 synchronized 와 비슷함)
- synchronized 와 달리 스레드를 블로킹하는 대신 코루틴 중단을 사용할 수 있다.
- 싱글스레드로 제한된 디스패처 사용보다 뮤텍스가 가볍고 성능이 좋다.

### 사용시 문제점
- lock 내부에 오래 걸리는 작업이 포함되면 Mutex 가 잠겨 오래걸릴 수 있다.
- 코루틴이 lock 을 두번 통과할 수 없기 때문에 교착상태(deadlock)에 빠져 블로킹 상태로 있을 수 있다.
  ```Kotlin
  suspend fun main() {
      val mutex = Mutex()
      mutex.withLock {
          mutex.withLock {
              print("실행 안됨")
          }
      }
  }
  ```

</br></br>

## 세마포어
- 하나의 접근만 허용하는 mutex 와는 달리 둘 이상이 접근할 수 있다.
- 공유 상태로 인해 생기는 문제는 해결할 수 없지만 동시 요청을 처리하는 수를 제한할 수 있다.
