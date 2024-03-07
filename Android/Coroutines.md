# Coroutines

## CoroutineContext
- Coroutine Builder, Scope, Continutation 모두 첫번째 파라미터는 `CoroutineContext` 를 쓰고 있습니다.
    
    ```kotlin
    public fun CoroutineScope.launch(
        context: CoroutineContext = EmptyCoroutineContext,
        start: CoroutineStart = CoroutineStart.DEFAULT,
        block: suspend CoroutineScope.() -> Unit
    ): Job {}
    ```
    ```kotlin
    public fun <T> CoroutineScope.async(
        context: CoroutineContext = EmptyCoroutineContext,
        start: CoroutineStart = CoroutineStart.DEFAULT,
        block: suspend CoroutineScope.() -> T
    ): Deferred<T> {}
    ```
    ```kotlin
    public interface CoroutineScope {
        public val coroutineContext: CoroutineContext
    }
    ```
    ```kotlin
    public inline fun <T> Continuation(
        context: CoroutineContext,
        crossinline resumeWith: (Result<T>) -> Unit
    ): Continuation<T> {}
    ```

### CoroutineContext 인터페이스
- CoroutineContext 는 코루틴의 동작을 정의하는 요소 집합입니다.
    - `Job` - 코루틴의 수명 주기를 제어합니다.
    - `CoroutineDispatcher` - 작업을 적절한 스레드로 디스패치 합니다.
    - `CoroutineName` - 디버깅에 유용한 코루틴 이름입니다.
    - `CoroutineExceptionHandler` - 포착되지 않은 예외를 처리합니다.
- 컨텍스트에서 모든 원소는 식별할 수 있는 유일한 key 를 가지고 있습니다. 키는 주소로 비교가 됩니다.
- 예를들어, `Job` 은 CoroutineContext 인터페이스를 구현한 CoroutineContext.Element 를 구현합니다.  
  ```kotlin
  public interface Job : CoroutineContext.Element
  ```  
- 다른 요소들도 동일하게 구성되어 있습니다.

### CoroutineContext에서 원소 찾기
- 컬렉션과 비슷하기 때문에 key 를 사용해서 원소를 찾을 수 있습니다.
  ```kotlin
  fun main() {
    val ctx: CoroutineContext = CoroutineName("A name")
  		
  	val coroutineName: CoroutineName? = ctx[CoroutineName]. // 혹은 ctx.get(CoroutineName)
  	val job: Job? = ctx[Job]
  		
  	// coroutineName 은 "A name"이 나오지만, job 은 설정된게 없기때문에 null
  }
  ```
- CoroutineName 은 타입이나 클래스가 아닌 Companion 객체입니다.
- 따라서, `ctx[ContextName]` 은  `ctx[ContextName.Key]` 가 됩니다.
    ```kotlin
    public data class CoroutineName(
      val name: String
    ) : AbstractCoroutineContextElement(CoroutineName) {
      public companion object Key : CoroutineContext.Key<CoroutineName>
      override fun toString(): String = "CoroutineName($name)"
    }
    ```
- Job 도 마찬가지로 Companion 객체가 있습니다.
    ```kotlin
    public interface Job : CoroutineContext.Element {
      public companion object Key : CoroutineContext.Key<Job>
      ...
    }
    ```

### Context 더하기
- CoroutineContext 는 서로 합쳐서 하나의 Context 로 만들 수 있습니다.    
    ```kotlin
    fun main() {
    	val ctx1 = CoroutineName("Name1")
    	val ctx2 = Job()
    		
    	val ctx3 = ctx1 + ctx2   // ctx1 의 CoroutineName 과 ctx2 의 Job 을 가진다.
    }
    ```
- 만약 동일한 원소가 더해지면 새로운 원소가 기존을 대체합니다.    
    ```kotlin
    fun main() {
      val ctx1 = CoroutineName("Name1")
    	val ctx2 = CoroutineName("Name2")
    		
    	val ctx3 = ctx1 + ctx2   // ctx3 의 CoroutineName 는 "Name2"가 된다.
    }
    ```

### 비어 있는 CoroutineContex
- `EmptyCoroutineContext` 를 사용 가능하고, 내부 원소들은 전부 null 입니다.

### 원소 제거
- `minusKey` 함수에 key 를 넣는 방식으로 원소를 컨텍스트에서 제거할 수 있습니다.    
    ```kotlin
    fun main() {
      val ctx1 = CoroutineName("Name1") + Job()
    		
    	val ctx2 = ctx1.minusKey(CoroutineName)  // ctx2 의 CoroutineName 은 제거되어 null이 된다.
    }
    ```

### Context folding
- `fold` 함수를 사용하여 원소들을 차례로 연산 한다.
- collection 의 fold 와 동일하다.

### CoroutineContext 와 Builder
- 부모-자식 관계의 영향중 하나로 부모는 기본적으로 컨텍스트를 자식에게 전달합니다.
- 자식은 부모로부터 컨텍스트를 상속받는다고 할 수 있습니다.
    ```kotlin
    fun main() = runBlocking(CoroutineName("main")) {
      // coroutineName == main
      val v1 = async {
    	  // coroutineName == main
        delay(1000)
        42
      }
      launch {
    	  // coroutineName == main
        delay(3000)
      }
      // coroutineName == main
    }
    ```
- 자식들은 인자에 정의된 특정 컨텍스트를 가질 수 있습니다.
- 전달받은 컨텍스트는 부모로부터 상속받은 컨텍스트를 대체합니다.
    ```kotlin
    fun main() = runBlocking(CoroutineName("main")) {
      // coroutineName == main
      val v1 = async(CoroutineName("inner1")) {
    	  // coroutineName == inner1
        delay(1000)
        42
      }
      // coroutineName == main
      launch(CoroutineName("inner2")) {
    	  // coroutineName == inner2
        delay(3000)
      }
      // coroutineName == main
    }
    ```
- CoroutineContext 계산 공식은 아래와 같습니다.    
    ```kotlin
    defaultContext + parentContext + childContext
    ```
- defaultContext 의 경우 Dispatcher 는 default 를 사용합니다.

### Suspend 함수에서 Context 접근하기
- suspend 함수에서도 동일하게 CoroutineContext 에 접근할 수 있습니다.    
    ```kotlin
    suspend fun test() = withContext(CoroutineName("outer")) {
      // coroutineName == outer
      launch(CoroutineName("inner")) {
      	// coroutineName == inner
      }
      // coroutineName == outer
    }
    ```
