---

title : "[Kotlin] Coroutines #1"
excerpt : "Coroutines basics을 공부해보자"

categories:
  - Android
tags :
  - Kotlin
 
---

매쉬업에 참여하면서 비동기 스터디를 진행하게 됐다. 

[코루틴 공식문서](https://kotlinlang.org/docs/coroutines-basics.html)를 보며 공부하기 ~ 

---

### 01. Coroutines basics

> **Thread.sleep 사용**

```java
import kotlinx.coroutines.*

fun main() {
    GlobalScope.launch {
        delay(1000) //1초 대기 후 
        println("World!!") //world 출력 
    }
    println("Hello ")
    Thread.sleep(2000) //main 스레드를 2초동안 blocking 시킴 
}
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/29348f80-6e78-4eb1-bd03-6d4db8949e31/스크린샷_2021-08-03_23.56.57.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/29348f80-6e78-4eb1-bd03-6d4db8949e31/스크린샷_2021-08-03_23.56.57.png)

GlobalScope라는 코루틴을 만들어서 블록 내부를 메인 스레드와 동시에 실행 

Thread.sleep(2000)으로 메인 스레드를 2초동안 블록킹 해놨기 때문에 "Hello"가 출력 된 후, 코루틴이 1초 대기 하고 Wrold가 출력됨 !

```java
import kotlinx.coroutines.*

fun main() {
    GlobalScope.launch {
        delay(3000) //3초 대기 후 
        println("World!!") //world 출력 
    }
    println("Hello ")
    Thread.sleep(2000) //main 스레드를 2초동안 blocking 시킴 
}
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f46492d9-02b3-4312-9a4f-085126f72b0b/스크린샷_2021-08-03_23.56.40.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f46492d9-02b3-4312-9a4f-085126f72b0b/스크린샷_2021-08-03_23.56.40.png)

마찬가지로 메인 스레드와 코루틴이 동시에 실행된다. 하지만 메인스레드를 2초 멈춰놓았기 때문에 3초를 대기해야하는 코루틴을 기다리지 않고 끝나버려서 World 가 출력되지 않는다. 

> **runBlocking 사용**

```java
import kotlinx.coroutines.*

fun main() {
    GlobalScope.launch {
        delay(1000)
        println("World!!")
    }
    println("Hello ")
    runBlocking { //Thread.sleep 대신 2초동안 main 스레드 blocking 
        delay(2000)
    }
}
```

Thread.sleep과 동일한 기능을 하는것이 runBlocking이라는 코루틴 빌더이다. 자신을 호출한 스레드를 blocking 한다. 

delay → suspend 코루틴 안에서만 수행해야함 (main이 중단되면 안되기때문에 밖에서 사용할 수 없음 , 명시적인 코루틴 블록을 써줘야함 ! ⇒ 너 멈춰! runBlocking { } )

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/047e8528-9184-4ece-a3bb-90ff6ca735cb/스크린샷_2021-08-04_00.00.40.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/047e8528-9184-4ece-a3bb-90ff6ca735cb/스크린샷_2021-08-04_00.00.40.png)

❌  주의해야하는 점 → delay함수는 suspend함수( 중단하는 함수) 기 때문에 메인스레드에서 단독으로 사용할 수 없고, 위에 나와있듯이 코루틴이나 또 다른 suspend 함수에서 사용해야한다. 

```java
import kotlinx.coroutines.*

fun main() = runBlocking {
    GlobalScope.launch {
        delay(1000)
        println("World!!")
    }
    println("Hello ")
    delay(2000)
}
```

blocking 하는것이 메인 스레드기 때문에 이렇게 써줘도 된다!

> **delay 없이 job을 만들어 기다리게 하기**

delay(숫자) 이렇게 사용하는것은 좋은 방법이 아니다. ex) 위의 3초 대기때문에 world가 출력되지 않는 예제 

우리의 목적은 코루틴 내부의 일을 끝내는것이기 때문에 코루틴 내부의 일이 끝날때까지 대기하기 위해서는.. 

→ launch 나 async가 반환하는  **job객체** 이걸 이용하자!

```java
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = GlobalScope.launch {
        delay(1000)
        println("World!!")
    }
    println("Hello ")
    job.join() //코루틴 내부의 일 끝날때까지 대기해 ! delay(숫자)가 아닌 job객체 사용  
}
```

 delay(숫자) 대신 job.join()을 사용했다. 

x초 동안 대기해 ! 라고 직접 정해주지 않아도, 코루틴 내부의 일이 끝날때까지 main스레드가 잘 blocking 된다. 

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/29348f80-6e78-4eb1-bd03-6d4db8949e31/스크린샷_2021-08-03_23.56.57.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/29348f80-6e78-4eb1-bd03-6d4db8949e31/스크린샷_2021-08-03_23.56.57.png)

> **structured concurrency**

앞선 예제들 보다 좀 더 좋은 방법 

delay로 몇초동안 대기할지 명시하지 않고 job.join()을 사용해서 일 끝날때까지 대기해! 라고 해줬다. 하지만 여러개의 코루틴을 사용하다 보면 job.join()을 여러개 써줘야하는 문제점(관리 불편)이 발생한다. 이걸 해결하려면, 서로 구조적으로 단계를 만들어서 기다리게 만들자 

```java
import kotlinx.coroutines.*

fun main() = runBlocking { //#1
    val job = GlobalScope.launch { // #2
        delay(1000)
        println("World!!")
    }
    println("Hello ")
    job.join() 
}
```

앞의 예제에서는 #1의 CoroutineScope와 #2의 CoroutineScope 두개가 만들어지고, 서로 아무런 연관성이 없어서 main스레드에서 대기해주지 않았다.

```java
import kotlinx.coroutines.*

fun main() = runBlocking { //#1 부모코루틴 
    launch { //#2 자식코루틴 
        delay(1000)
        println("World!!")
    }
    println("Hello ")
}
```

하지만 새로운 top-level scope를 만들어주지 않고, #1의 자식으로 코루틴을 만들면, #1은 #2의 부모코루틴이 되기때문에 자식 코루틴이 완료될때까지 자동으로 기다려준다. (thread.sleep이나 delay, job.join 없이) 

> **Extract function refactoring**

world를 찍는 부분을 따로 함수로 빼서 호출해보자 

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/144f45fc-7ec7-4cc7-b617-5a77f5fb817b/스크린샷_2021-08-04_00.25.21.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/144f45fc-7ec7-4cc7-b617-5a77f5fb817b/스크린샷_2021-08-04_00.25.21.png)

 아까 떴던 오류가 뜬다 ㅠ printWorld는 일반 함수라서 suspend 키워드를 붙여주어 일시중지가 될 수 있다는 함수라는것을 알려줘야한다. 

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1c8fe87e-02af-4174-b56a-c3b2f1b489a2/스크린샷_2021-08-04_00.26.36.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1c8fe87e-02af-4174-b56a-c3b2f1b489a2/스크린샷_2021-08-04_00.26.36.png)

오류가 사라진 모습을 볼 수 있다. ㅎㅎ 

 

> **Coroutines ARE light-weight**

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking { //#1 부모코루틴 
    launch { //#2 자식코루틴 
        delay(1000)
        println("World!!")
    }
    println("Hello ")
}
```

많은 양의 코루틴을 만들어 10만개의 점을 찍는 예제지만, Thread로 돌릴때보다 훨씬 덜 버벅거리고 빠르다.

---

**참고**

[https://kotlinlang.org/docs/coroutines-basics.html]

[https://www.youtube.com/watch?v=14AGUuh8Bp8&list=PLbJr8hAHHCP5N6Lsot8SAnC28SoxwAU5A&index=2]
