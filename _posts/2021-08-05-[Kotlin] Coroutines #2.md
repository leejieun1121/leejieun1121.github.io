---

title : "[Kotlin] Coroutines #2"
excerpt : "Cancellation and Timeouts을 공부해보자"

categories:
  - Android
tags :
  - Kotlin
 
---

> 코루틴 취소

실행중인 코루틴을 취소하기 위해서는 2가지 방식이 있다.

1. 주기적으로 suspend 함수를 호출 → 재개 될때 cancel 됐는지 확인해서 exception을 던지는 방식
2. 명시적으로 상태를 체크 (isActive) 해서 코루틴을 종료시킬지말지 파악 하는 방식 
3. TimeOut

> Cancelling coroutine execution

코루틴을 취소시키는 방법중 첫번째 방법이다. 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = launch {
        repeat(1000){  i ->
            println("job: I'm sleeping $i...")
            delay(500)
        }
    }
    delay(1300) //1.3초 뒤에 
    println("main: I'm tired of waiting!")
    job.cancel() //코루틴을 종료해라 
    job.join()
    println("main: Now I can quit.")
}
```

![image](https://user-images.githubusercontent.com/53978090/128207627-c6ccca44-50fe-4282-b2dc-dd52654955f0.png)

1.3초 뒤, job.cancel()함수를 통해 실행중이던 코루틴이 중단되었다.

(job.join()을 job.cancel뒤에 써야함 안그러면 cancel되지 않고 계속 기다림 )

> Cancellation is cooperative

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val startTime = System.currentTimeMillis()
    val job = launch(Dispatchers.Default) {
        var nextPrintTime = startTime
        var i = 0
        while(i < 5){
            if(System.currentTimeMillis() >= nextPrintTime) {
                println("job: I'm sleeping ${i++} ...")
                nextPrintTime += 500
            }
        }
    }

    delay(1300)
    println("main: I'm tired of waiting!")
    job.cancelAndJoin()
    println("main: Now I can quit.")
}
```

![image](https://user-images.githubusercontent.com/53978090/128207661-566a4f9c-22d6-4f8c-a60f-489d3743cc72.png)

위의 예제와는 다르게 job이 cancel 되지 않고 계속해서 출력됨 

처음에는 job.cancelAndJoin()이 무슨 역할을 하는줄 알았지만 얘는 단순히 

```kotlin
public suspend fun Job.cancelAndJoin(){
	cancel()
	retrun join()
}
```

이렇게 생긴 함수로 cancel 다음 join을 해주는 함수이다. 

그럼 왜 취소되지 않았을까 ? 

그 이유는 !  → 코루틴 내부에  suspend 호출을 하지 않았기 때문 

 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = launch {
        repeat(1000){  i ->
            println("job: I'm sleeping $i...")
            delay(500)
        }
    }
    delay(1300) //1.3초 뒤에 
    println("main: I'm tired of waiting!")
    job.cancel() //코루틴을 종료해라 
    job.join()
    println("main: Now I can quit.")
}
```

앞의 예제에는 코루틴 내부에 delay(500)이라는 suspend 함수(중단 함수)가 있어 job.cancel()을 호출하면 1.3초 뒤에 코루틴이 중단 되었었다. 

하지만 위의 예제에서는 단순연산만 존재하고 suspend 함수가 존재하지 않는다. 코루틴을 취소하기 위해서 코루틴 내부에 delay를 걸어주자.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val startTime = System.currentTimeMillis()
    val job = launch(Dispatchers.Default) {
        var nextPrintTime = startTime
        var i = 0
        while(i < 5){
            if(System.currentTimeMillis() >= nextPrintTime) {
                delay(1)
                println("job: I'm sleeping ${i++} ...")
                nextPrintTime += 500
            }
        }
    }

    delay(1300)
    println("main: I'm tired of waiting!")
    job.cancelAndJoin()
    println("main: Now I can quit.")
}
```

![image](https://user-images.githubusercontent.com/53978090/128207705-df3f702c-fbef-43f3-9d46-341563753d7b.png)

delay함수를 추가해주고 나니까 원하는 대로 1.3초 뒤에 코루틴이 취소된다. 

delay(1) 과 같은 역할을 하는 메서드가 **yield()**메서드다. 

- yield()

    : 해당 위치에서 코루틴을 일시중단함 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val startTime = System.currentTimeMillis()
    val job = launch(Dispatchers.Default) {
        var nextPrintTime = startTime
        var i = 0
        while(i < 5){
            if(System.currentTimeMillis() >= nextPrintTime) {
                yield() //delay(1) -> yield() 
                println("job: I'm sleeping ${i++} ...")
                nextPrintTime += 500
            }
        }
    }

    delay(1300)
    println("main: I'm tired of waiting!")
    job.cancelAndJoin()
    println("main: Now I can quit.")
}
```

![image](https://user-images.githubusercontent.com/53978090/128207736-fb9d8fe2-de01-4657-986e-705a1f9003df.png)

delay(1)을 yield()로 바꿔주었더니 똑같은 결과가 나왔다. 

그렇다면 코루틴 내부에 delay나 yield 함수를 써서 코루틴이 취소되는 원리는 뭘까 ?

→ job.cancel()을 호출하면 코루틴 내부에서 suspend가 되었다가 다시 재개되는 시점에  suspend함수가 exception을 던지기 때문이다. 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val startTime = System.currentTimeMillis()
    val job = launch(Dispatchers.Default) {
        try{
            var nextPrintTime = startTime
            var i = 0
            while(i < 5){
                if(System.currentTimeMillis() >= nextPrintTime) {
                    yield()
                    println("job: I'm sleeping ${i++} ...")
                    nextPrintTime += 500
                }
            }
        }catch (e: Exception){
            println("Exception: $e")
        }

    }

    delay(1300)
    println("main: I'm tired of waiting!")
    job.cancelAndJoin()
    println("main: Now I can quit.")
}
```

![image](https://user-images.githubusercontent.com/53978090/128207765-15e03319-153b-46cd-97cd-187060107b6b.png)

suspend함수가 Exception을 던지는지 확인하기 위해 코루틴 내부를 try-catch문으로 감싸고 예외를 출력해봤더니 JobCancellationException 을 발생시켜 코루틴이 종료되는것을 알 수 있다. ⇒ 코루틴 스스로가 협조적임! (suspend함수를 사용해서 자신이 cancel을 체크!)

> Making computation code cancellable

코루틴을 취소시키는 방법중 2번째 방법이다. 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val startTime = System.currentTimeMillis()
    val job = launch(Dispatchers.Default) {
        try {
            var nextPrintTime = startTime
            var i = 0
            println("isActive 상태: $isActive")
            while (isActive) {
                if (System.currentTimeMillis() >= nextPrintTime) {
                    println("job: I'm sleeping ${i++} ...")
                    nextPrintTime += 500
                }
            }
            println("isActive 상태: $isActive")
        } catch (e: Exception) {
            println("Exception : $e")
        }
    }

    delay(1300)
    println("main: I'm tired of waiting!")
    job.cancelAndJoin()
    println("main: Now I can quit.")
}
```

![image](https://user-images.githubusercontent.com/53978090/128207787-1b2713f8-6ad4-4bfc-9d1c-aec1e75a174e.png)

코루틴 내부에 suspend함수를 사용하지 않았는데 isActive의 상태가 바뀌면서 1.3초뒤에 종료되었다. 

이전에 suspend함수를 사용하는 방식과는 다르게 취소될때 exception을 날리지 않는다. 

isActive는 코루틴의 확장 프로퍼티로 다음과 같이 생긴 함수이며, 코루틴 Job이 실제로 종료되었는지 체크한다! 

```kotlin
@Suppress("EXTENSION_SHADOWED_BY_MEMBER")
public val CoroutineScope.isActive: Boolean
    get() = coroutineContext[Job]?.isActive ?: true
```

> Closing resources with finally

: 코루틴을 종료할때 resources 해제를 하면 위치를 기억해야한다. 

ex) 파일, db등 처리중일때 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = launch {
        try {
            repeat(1000){   i ->
                println("job: I'm sleeping $i ...")
                delay(500)
            }
        } finally {
						//resources 해제 코드 작성 ! 
            println("job: I'm running finally")
        }
    }
    delay(1300)
    println("main: I'm tired of waiting!")
    job.cancelAndJoin()
    println("main: Now I can quit.")
}
```

![image](https://user-images.githubusercontent.com/53978090/128207811-fd76603f-5363-4e6f-9ed5-87e225636c75.png)

코루틴 내에 delay함수 (suspend함수)가 존재하기 때문에 취소에 협조적이다. 그러므로 job.cancelAndJoin()을 호출하면 중지되었다가 재개될때 exception을 던지며 finally 코드가 실행되고 여기서 resources 해제 코드를 작성하면 된다. 

> Run non-cancellable bloc

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = launch {
        try {
            repeat(1000){   i ->
                println("job: I'm sleeping $i ...")//#1 
                delay(500)
            }
        } finally {
            withContext(NonCancellable){
                println("job: I'm running finally")
                delay(1000)
                println("job: And I've just delayed for 1 sec because I'm non-cancellable")
            }
        }
    }
    delay(1300)
    println("main: I'm tired of waiting!")
    job.cancelAndJoin()
    println("main: Now I can quit.")
}
```

![image](https://user-images.githubusercontent.com/53978090/128207827-98bb68e4-dd99-4830-a177-758386c5ca2c.png)

job.cancelAndJoin()함수 호출로 인해 코루틴이 중단되고 finally 블록으로 빠졌지만, 다시 여기서 코루틴이 실행된다. 

> Timeout

실행중인 job을 cancel하는것이 아니라, 이 시간이 지나면 해당 코루틴은 취소된다~ 라고 지정하는 방식

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    withTimeout(1300){
        repeat(1000){ i ->
            println("I'm sleeping $i ...")
            delay(500)
        }
    }
}
```

![image](https://user-images.githubusercontent.com/53978090/128207839-92f9b023-4941-4911-bd79-6988d7c0e5c1.png)

delay()나 yield(), isActive를 사용하지 않고 withTimeout(시간) 을 사용해서 해당 시간이 지나면 코루틴이 바로 취소된다. 하지만 여기서는 TimeoutCancellationException이 나타난다.

→ 따로 코루틴을 만들지 않고 runBlocking을 사용하여 main에서 작성했기 때문에 발생함 

> withTimeoutOrNull

위의 문제를 처리하기 위해 사용한다. 예외가 발생했을때, exception을 던지는것이 아니라 null을 반환하게 된다. 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val result = withTimeoutOrNull(1300){
        repeat(1000){ i ->
            println("I'm sleeping $i ...")
            delay(500)
        }
        "Done"
    }
    println("Result: $result")
}
```

![image](https://user-images.githubusercontent.com/53978090/128207852-3c6d807c-ecbf-44be-8afd-4d4c2b0642e6.png)

앱이 죽지 않고 결과 null을 출력하는것을 볼 수 있다!  

---

**참고**

[[https://kotlinlang.org/docs/coroutines-basics.html](https://kotlinlang.org/docs/coroutines-basics.html)]

[[https://www.youtube.com/watch?v=14AGUuh8Bp8&list=PLbJr8hAHHCP5N6Lsot8SAnC28SoxwAU5A&index](https://www.youtube.com/watch?v=14AGUuh8Bp8&list=PLbJr8hAHHCP5N6Lsot8SAnC28SoxwAU5A&index=2)=3]
