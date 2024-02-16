# Data class
toString(), hashCode(), equals(), copy() 메서드를 자동으로 생성한다.

## 특징
- 상속 받을 수 없음.
- 1개 이상의 프로퍼티를 가지고 있어야 함.

### 문자열 표현 : toString()
- 코틀린에서 기본 제공도는 객체의 문자열 표현은 Client@1q2w3e 같은 방식이다.  
- 이 기본 구현을 바꾸려면 아래와 같이 toString 메서드를 오버라이드하여 사용한다.
```kotlin
class Client(val name: String, val phoneNumber: Int) {
  override fun toString() = "Client (name=$name, phoneNumber=$phoneNumber)"
}
```

### 객체의 동등성 : equals()
- 코틀린에서는 == 연산자를 통해 equals 를 호출해서 객체를 비교한다.
- 주소가 같은지 확인하기 위해선 === 연산자를 사용한다.

### 해시 컨테이너 : hashCode()
- 자바에서는 equals 를 오버라이드할 때 반드시 hashCode 도 함께 오버라이드 해야 한다.
- JVM 언어에서는 equals 가 true 를 반환하는 두 객체는 반드시 같은 hashCode 를 반환해야 한다 라는 제약이 있다.

### copy()
- 데이터 클래스의 모든 프로퍼티를 읽기 전용으로 만들어 `immutable class`로 만들라고 권장한다.
- copy() 메서드는 객체를 복사하면서 일부 프로퍼티를 바꿀 수 있게 해주는 메서드다.
