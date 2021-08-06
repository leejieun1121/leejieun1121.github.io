---

title : "[Kotlin] Coroutines #3"
excerpt : "Composing Suspending Functions 을 공부해보자"

categories:
  - Android
tags :
  - Kotlin
 
---

> Sequential by default

코루틴은 순차적으로 동작한다. 

```kotlin
import kotlinx.coroutines.*
import kotlin.system.measureTimeMillis

fun main() = runBlocking {
    val time = measureTimeMillis {
        val one = doSomethingUsefulOne()
        println("첫번째 함수 호출 끝")
        val two = doSomethingUsefulTwo()
        println("두번째 함수 호출 끝")
        println("The answer is ${one + two}")
    }
    println("Completed in $time ms")
}

suspend fun doSomethingUsefulOne() : Int {
    println("첫번째 함수 호출")
    delay(1000) //대체로 여기서 retrofit 호출 등 주요 작업 실행
    return 13
}

suspend fun doSomethingUsefulTwo() : Int {
    println("두번째 함수 호출")
    delay(1000)
    return 29
}
```

![image](https://user-images.githubusercontent.com/53978090/128464003-c2d2d523-959a-424e-9402-d3f39e7d3e08.png)

첫번째 작업 걸린 시간과 두번째 작업 걸린 시간을 합쳐 출력하는 예제이다. 비동기 함수를 호출한 순서대로 동작하는걸 볼 수 있다. 각각 함수마다 1000ms씩 delay를 걸어줬기 때문에 총 2014ms가 걸렸다. 

> Concurrent using async

위의 예제에서는 첫번째 연산이 다 끝난 후, 두번째 연산이 실행되었다. 근데 만약 작업 one과 two가 순차적으로 실행되는 일(예를 들어, one = 서버에서 데이터를 가져오는 작업 two = 가져온 데이터를 가공하는 작업) 이 아니라면(= 각 작업끼리 의존성이 없는 경우) 하나의 작업이 끝날때까지 기다리지 않고 두개의 작업을 동시에 실행시키는게 더 효율적이다. 

```kotlin
import kotlinx.coroutines.*
import kotlin.system.measureTimeMillis

fun main() = runBlocking {
    val time = measureTimeMillis {
        val one = async { doSomethingUsefulOne() }
        println("첫번째 함수 호출 끝")
        val two = async { doSomethingUsefulTwo() }
        println("두번째 함수 호출 끝")
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")
}

suspend fun doSomethingUsefulOne() : Int {
    println("첫번째 함수 호출")
    delay(1000) 
    return 13
}

suspend fun doSomethingUsefulTwo() : Int {
    println("두번째 함수 호출")
    delay(1000)
    return 29
}
```

![image](https://user-images.githubusercontent.com/53978090/128464018-244917cd-f52b-4b14-ad98-f3090448290c.png)

async를 사용하지 않은 앞의 예제에서는 첫번째 호출 → 첫번째 호출 끝 → 두번째 호출 → 두번째 호출 끝 이런 순서로 동작 되었었다.

suspend 함수를 실행하는 동안 중지되었다가 재개될때 다시 실행되어 이렇게 동작했던 것인데, async를 사용해서 호출하게 되면 해당 코루틴 블럭이 대기하지 않고 그 다음 코드를 바로 실행하고 await()으로 끝날때까지 기다렸다가 두개의 작업이 둘다 끝났을때 출력된다. 아까 2014ms 가 걸렸던 작업이 1027ms로 절반 단축되었다. 

하지만 async를 사용해도 순차적으로 실행시키는 방법이 있다. 첫번째 작업에await()을걸어 끝날때까지 기다린 후, 두번째 작업을 실행시키면 된다.

```kotlin
import kotlinx.coroutines.*
import kotlin.system.measureTimeMillis

fun main() = runBlocking {
    val time = measureTimeMillis {
        val one = async { doSomethingUsefulOne() }
        val oneDelay = one.await()
        println("첫번째 함수 호출 끝")
        val two = async { doSomethingUsefulTwo() }
        println("두번째 함수 호출 끝")
        println("The answer is ${oneDelay + two.await()}")
    }
    println("Completed in $time ms")
}

suspend fun doSomethingUsefulOne() : Int {
    println("첫번째 함수 호출")
    delay(1000) //대체로 여기서 retrofit 호출 등 주요 작업 실행
    return 13
}

suspend fun doSomethingUsefulTwo() : Int {
    println("두번째 함수 호출")
    delay(1000)
    return 29
}
```

![image](https://user-images.githubusercontent.com/53978090/128464035-10babe87-9ffd-40a1-9e7c-b2bd5083eb82.png)

one.await()을 사용해서 첫번째 함수가 끝날때까지 기다리고 두번째 함수를 호출했기 때문에 async를 사용하기 전처럼 2025ms가 나온것을 확인할 수 있다. 

> Lazily started async

코루틴 실행을 늦추기 위해 optional 파라미터인 start의 값을 CorotineStart.LAZY로 설정 (아무것도 설정하지 않으면 CorotineStart.Default)

```kotlin
import kotlinx.coroutines.*
import kotlin.system.measureTimeMillis

fun main() = runBlocking {
    val time = measureTimeMillis {
        val one = async(start = CoroutineStart.LAZY) { doSomethingUsefulOne() }
        val two = async(start = CoroutineStart.LAZY) { doSomethingUsefulTwo() }
        one.start()
        two.start()
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")
}

suspend fun doSomethingUsefulOne() : Int {
    delay(1000)
    return 13
}

suspend fun doSomethingUsefulTwo() : Int {
    delay(1000)
    return 29
}
```

![image](https://user-images.githubusercontent.com/53978090/128464049-3b8cf02b-cdab-4528-a859-7937f8e31302.png)

시작을 LAZY로 설정했지만 one.start()와  two.start()를 호출해서 바로바로 작업이 실행되었기 때문에 동시에 작업한것과 같은 결과가 나왔다. 하지만 start를 호출하지 않는다면..?

```kotlin
import kotlinx.coroutines.*
import kotlin.system.measureTimeMillis

fun main() = runBlocking {
    val time = measureTimeMillis {
        val one = async(start = CoroutineStart.LAZY) { doSomethingUsefulOne() }
        val two = async(start = CoroutineStart.LAZY) { doSomethingUsefulTwo() }
//        one.start()
//        two.start()
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")
}

suspend fun doSomethingUsefulOne() : Int {
    delay(1000)
    return 13
}

suspend fun doSomethingUsefulTwo() : Int {
    delay(1000)
    return 29
}
```

![image](https://user-images.githubusercontent.com/53978090/128464059-05e5ffe9-9916-46ae-b932-764e38422f64.png)

아까와 똑같이 async와 await를 사용했지만, 두개의 작업을 따로따로 실행한 결과가 나왔다.(start = CoroutineStart.LAZY 걸어놓은 상태)

→ 그 이유는, 각각 one two 코루틴을 만들었지만 start()를 호출하지 않았기 때문에  one.await()을 만날때까지 실행되지 않고,await()을 호출했을때 첫번째 코루틴이 실행된다. 그리고 첫번째 코루틴이 끝날때까지 기다린 뒤 two.await()을 만나 두번째 코루틴이 실행되고 끝나기 때문에 2023ms 가 걸린다. 

> Async-style functions

GlobalScope를 사용해서 코루틴을 일반 함수로 만들어 어디서든지 사용할 수 있게 만든 예제이다. 

```kotlin
import kotlinx.coroutines.*
import kotlin.system.measureTimeMillis

fun main() {
    try{
        val time = measureTimeMillis {
            val one = somethingUsefulOneAsync()
            val two = somethingUsefulTwoAsync()

            println("예외 발생")
            throw Exception("Exception")

            runBlocking { //값을 얻어오기 위해 메인스레드 blocking 
                println("The answer is ${one.await() + two.await()}")
            }
        }
        println("Completed in $time ms")
    }catch (e: Exception){
        println("Exception : $e")
    }

    runBlocking {
        delay(100000)
    }
}

fun somethingUsefulOneAsync() = GlobalScope.async {
    println("첫번째 함수 호출")
    val res = doSomethingUsefulOne()
    println("첫번째 함수 호출 끝")
    res
}
fun somethingUsefulTwoAsync() = GlobalScope.async {
    println("두번째 함수 호출")
    val res = doSomethingUsefulTwo()
    println("두번째 함수 호출 끝")
    res
}

suspend fun doSomethingUsefulOne() : Int {
    delay(1000)
    return 13
}

suspend fun doSomethingUsefulTwo() : Int {
    delay(1000)
    return 29
}
```

![image](https://user-images.githubusercontent.com/53978090/128464076-ac65acb6-9aa6-4d4a-a1bb-ce30aa816d5b.png)

위의 예제에서는 somethingUsefulOneAsync과 somethingUsefulTwoAsync함수를 GlobalScope를 이용해서 만들었기때문에 suspend 함수가 아니고, 언제 어디서든지 호출할 수 있다. 

async와 await사이에 예외가 발생했으면 코루틴이 중단되는게 맞지만, 위의 예제에서는 예외가 발생했지만 코루틴이 중지되지 않고 계속 실행되는것을 볼 수 있다.  GlobalScope에서 실행되었기 때문에 전체영역에서 동작하게 되고, 저기에서 예외가 발생해도 어떠한 영향도 미치지 않기 때문!  

⇒ **GlobalScope에서 독립적인 형태로 코루틴을 실행시키면 안된다 → 예외처리 능력이 떨어진다.** 

> Structured concurrency with async

위의 방법은 예외처리 능력이 떨어지기 때문에, 권장하지 않고 coroutineScope안에서 suspend함수를 적절하게 사용해야 예외처리가 제대로 이루어질 수 있다. 

```kotlin
import kotlinx.coroutines.*
import kotlin.system.measureTimeMillis

fun main() = runBlocking {
    try{
        val time = measureTimeMillis {
            println("The answer is ${concurrentSum()}")
        }
        kotlin.io.println("Completed in $time ms")
    }catch (e:Exception){
        println("Exception : $e")
    }
}

suspend fun concurrentSum() : Int = coroutineScope {  
    val one = async { doSomethingUsefulOne() }
    val two = async { doSomethingUsefulTwo() }

    delay(10)

    println("예외 발생")
    throw Exception("Exceptions")

    one.await() + two.await()
}

suspend fun doSomethingUsefulOne() : Int {
    println("첫번째 함수 호출")
    delay(1000) //여기서 exception을 알아차리고 cancel함 
    println("첫번째 함수 호출 끝")
    return 13
}

suspend fun doSomethingUsefulTwo() : Int {
    println("두번째 함수 호출")
    delay(1000)// 위와 동일 
    println("두번째 함수 호출 끝")
    return 29
}
```

![image](https://user-images.githubusercontent.com/53978090/128464093-5b924a23-af83-41eb-bc33-eae3b66c68ba.png)

concurrentSum함수는 GlobalScope가 아닌 coroutineScope기때문에 언제 어디서든지 불릴 수 있는 일반 함수가 아니라 suspend함수이다. (코루틴 안에서만 사용!)  그리고 concurrentSum함수는 coroutineScope로 설정했기 때문에 부모 Scope가 되고 doSomethingUsefulOne, doSomethingUsefulTwo가 각각 자식 코루틴이 된다. 자식 scope의 delay에서 예외가 발생한걸 알아차리기 때문에 다른 자식 코루틴도 자연스럽게 취소된다. ⇒ 전체 코루틴이 취소된다.

> Cancellation propagated coroutines hierarchy

```kotlin
import kotlinx.coroutines.*
import java.lang.ArithmeticException

fun main() = runBlocking<Unit> { //#1
   try{
       failedConcurrentSum()
   }catch (e : ArithmeticException){
       println("Computation failed with ArithmeticException")
   }
}

suspend fun failedConcurrentSum() : Int = coroutineScope {
    val one = async{ //#2
        try{
            delay(Long.MAX_VALUE)
            42
        } finally {
            println("First child was cancelled")
        }
    }

    val two = async<Int>{ //#3
        println("Second child throws an exception")
        throw ArithmeticException()
    }

    one.await() + two.await()
}
```

![image](https://user-images.githubusercontent.com/53978090/128464115-e815004d-8570-4f1c-b572-d45bf1e57a90.png)

#3 코루틴에서 exception이 발생하면, #2코루틴에서 try-catch에 걸려 finally블록이 호출되고, #1코루틴의 try-catch가 실행되어 예외를 출력한다. ⇒ 실행되던 중에 종료가 되면서 모든 resource를 해제하게 된다. 

Structured concurrency를 이용하는것이 아주 중요하다!! 

---

**참고**

<[https://tourspace.tistory.com/152](https://tourspace.tistory.com/152)>

<[https://kotlinlang.org/docs/composing-suspending-functions.html#structured-concurrency-with-async](https://kotlinlang.org/docs/composing-suspending-functions.html#structured-concurrency-with-async)>

<[https://www.youtube.com/watch?v=0viswXto028&list=PLbJr8hAHHCP5N6Lsot8SAnC28SoxwAU5A&index=4](https://www.youtube.com/watch?v=0viswXto028&list=PLbJr8hAHHCP5N6Lsot8SAnC28SoxwAU5A&index=4)>
