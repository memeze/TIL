# 코루틴 스코프 함수

</br>

## 코루틴 스코프 함수가 소개되기 전에 사용했던 방법들

1. suspend 함수에서 suspend 함수를 호출한다.
    - 문제는 작업이 동시에 이루어지지 않음 (작업시간 1초인 작업이 5개 실행되면 5초 소요)

1. 두개의 suspend 함수를 동시에 실행하기위해 `GlobalScope.async` 를 사용한다.
    - 이때 **GlobalScope** 를 사용하면 부모 코루틴과 관계가 없어짐
    - 메모리 누수가 발생할 수 있고 쓸데없이 CPU 를 낭비함
    - 테스트하기가 어려움
    
1. scope 를 인자로 넘겨받아 (혹은 확장함수로 구성하여) 처리한다.
    - 2번보다 테스트가 용이한 방식
    - (SupervisorJob 미사용 가정하에) async 에서 예외가 발생하면 스코프가 닫힘
    - 잠재적인 위험이 있을 수 있음

</br></br>

## coroutineScope

```kotlin
suspend fun <R> coroutineScope(
    block: suspend CoroutineScope.() -> R
): R
```

- coroutineScope 는 스코프를 시작하는 suspend 함수이다.
- 인자로 들어온 함수(block)가 생성한 값을 반환한다.
- launch, async 와 다르게 리시버 없이 곧바로 호출한다.
- 새로운 코루틴을 생성하지만 새로운 코루틴이 끝날 때까지 중단한다.
    
    ```kotlin
    fun main() = runBlocking {
        val a = coroutineScope {
            delay(1000)
            10
        }
        
        println("a is calculated")
    		
        val b = coroutineScope {
            delay(1000)
            20
        }
    		
        println(a)
        println(b)
    }
    
    // (1초후)
    // a is calculated
    // (1초후)
    // 10
    // 20
    ```
    
- 생성된 스코프는 바깥 스코프에서 `coroutineContext` 를 상속받지만 Job을 오버라이딩 한다.
- 따라서 아래의 책임을 모두 수행한다.
    - 부모로부터 context 를 상속받는다.
    - 자신의 작업을 끝내기 전까지 모든 자식을 기다린다.
    - 부모가 취소되면 자식들 모두를 취소한다.

</br></br>

## 코루틴 스코프 함수

- `supervisorScope` : Job 대신 SupervisorJob 으로 사용되는 coroutineScope
- `withContext` : 코루틴 컨텍스트를 변경할 수 있는 coroutineScope
- `withTimeout` : 타임아웃이 있는 coroutineScope

### 코루틴 빌더와 코루틴 스코프의 차이

|  | 코루틴 빌더 (runBlocking 제외) | 코루틴 스코프 함수 |
| --- | --- | --- |
| 종류 | launch, async, produce | coroutineScope, supervisorScope, withContext, withTimeout |
| 구현 | CoroutineScope 확장함수 | Suspend 함수 |
| Context | CoroutineScope 리시버의 코루틴 컨텍스트 사용 | Suspend 컨티뉴에이션 객체의 코루틴 컨텍스트 사용 |
| 에러 처리 | Job 을 통해 부모로 전파 | 일반 함수와 동일하게 처리 |
| 실행 | 비동기인 코루틴 시작 | 코루틴 빌더가 호출된 곳에서 코루틴 시작 |

</br></br>

## withContext

- coroutineScope 와 비슷하지만 스코프의 컨텍스트를 변경할 수 있다.
- 코루틴 빌더와 같은 방식으로 부모 스코프의 컨텍스트를 대체한다.
- withContext(EmptyCoroutineContext) 와 coroutineScope() 는 같은 방식으로 동작한다.

</br></br>

## supervisorScope

- coroutineContext 의 Job 을 SupervisorJob 으로 오버라이딩하여 사용한다.
- withContext(SupervisorJob()) 은 supervisorScope 와 다르게 동작한다.
    - withContext 코루틴은 그대로 Job 을 사용하고 SupervisorJob 이 해당 잡의 부모가 된다.

</br></br>

## withTimeout

- withTimeout 의 인자로 타임아웃 값을 넣어준다.
- 실행시 지정한 시간보다 오래 걸리면 람다식은 취소된다. (TimeoutCancellationException)
- 테스트할 때 유용하게 사용된다.
- CancellationException 이기 때문에 부모까지만 취소되고 그 이상으로 전파되지 않는다.
- `withTimeoutOrNull` 을 사용하면 타임아웃시 람다가 취소되고 null 을 반환단다.

</br></br>

## 코루틴 스코프 함수 연결하기

- 코루틴 스코프 함수에서 다른 기능을 가지는 코루틴 스코프 함수를 호출한다.
    
    ```kotlin
    suspend fun test(): Test? = 
        withContext(Dispatchers.Default) {
            withTimeoutOrNull(1000) {
              calculate()
            }
        }
    ```

</br></br>

## 추가적인 연산

- coroutineScope 내에서 실행된 작업이 끝나는걸 기다릴 필요가 없을 경우,  
새로운 CoroutineScope 를 만들어서 주입시켜 처리할 수 있다.
