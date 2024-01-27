# 자바의 동등성

- 자바에서는 원시타입을 비교하기 위해 `==` 를 사용한다.
- `==` 는 두 피연산자의 값이 같은지 비교한다. (동등성)
    
    ```java
    int a = 1
    int b = 2
    
    System.out.println(a == b)  // false
    ```
    
- 참조 타입인 두 피연산자 사이에 `==` 를 사용할 경우, 주소값으로 비교한다.
- 두 피연산자의 주소값이 같은 곳을 가리키고 있다면 true 를 반환하는 것이다.
- String 은 원시타입이 아닌 참조 타입이기 때문에, 겉으로는 같아도 주소값이 다를 경우 false 다.
    
    ```java
    String a = new String("hi");  // 주소값: 1
    String b = new String("hi");  // 주소값: 2
    
    System.out.println(a == b);   // false
    ```
    
- 따라서, 자바에서는 두 객체(참조타입)의 동등성을 알기 위해서 `equals` 를 사용한다.
    
    ```java
    String a = new String("hi");      // 주소값: 1
    String b = new String("hi");      // 주소값: 2
    
    System.out.println(a.equals(b));  // true
    ```

---
### Java String pool
> String 은 두 가지 생성 방식이 있고 각각의 차이점이 존재한다.

#### new 연산자를 이용한 방식
- new를 통해 String을 생성하면 Heap 영역에 존재하게 된다.

#### 리터럴을 이용한 방식
- string constant pool이라는 영역에 존재하게 된다.
  
여기서 그냥 `“hi”` 를 쓰게되면 String pool 을 사용해서 같은 리터럴을 공유하여 같은 주소값을 참조하기 때문에, 객체를 새로 생성한다.
