---

title : "[Kotlin] Coroutines #4"
excerpt : "Coroutine Context and Dispatchers 을 공부해보자"

categories:
  - Android
tags :
  - Kotlin
 
---

> Dispatchers and threads

코루틴은 coroutineContext에서 실행됨 

**Dispatchers → Context의 요소이며** **어떤 스레드나 어떤 스레드풀에서 실행될지 결정해줌** 

코루틴을 빌드할때 코루틴 빌더(launch, async를 사용하는데, option으로 context parameter를 가지고 있음 → 어떤 디스패처를 쓸건지 정할 수 있다. 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {//#0 이거포함 5개
    //4개의 코루틴을 실행 !
    launch { //#1
        println("main runBlocking : " + "I'm working in thread ${Thread.currentThread().name}")
    }

    launch(Dispatchers.Unconfined){//#2
        println("Unconfined  : " + "I'm working in thread ${Thread.currentThread().name}")
    }

    launch(Dispatchers.Default){//#3
        println("Default  : " + "I'm working in thread ${Thread.currentThread().name}")
    }

    launch(newSingleThreadContext("MyOwnThread")){//#4 
        println("newSingleThreadContext  : " + "I'm working in thread ${Thread.currentThread().name}")
    }
}
```

![image](https://user-images.githubusercontent.com/53978090/128694261-94711500-72af-402c-baa4-8cdc7134ee55.png)

#1의 **launch는 파라미터가 없기때문에** **#0 runBlocking으로부터 상속받은 자식 코루틴이 됨** → 컨텍스트를 그대로 이용함! 그래서 main에서 실행된다. 

#2에서는 **Dispatchers.Unconfine**d라는 파라미터를 사용했다. 이 디스패처는 **suspend point를 만날때까지 호출 스레드에서 코루틴을 시작하고, 재개될때는 중단 함수를 재개한 스레드에서 수행된다.** → 특정 스레드에 한정된 작업이 아닌경우 사용 [참고]<[https://myungpyo.medium.com/코루틴-공식-가이드-자세히-읽기-part-5-62e886f7862d](https://myungpyo.medium.com/%EC%BD%94%EB%A3%A8%ED%8B%B4-%EA%B3%B5%EC%8B%9D-%EA%B0%80%EC%9D%B4%EB%93%9C-%EC%9E%90%EC%84%B8%ED%9E%88-%EC%9D%BD%EA%B8%B0-part-5-62e886f7862d)>

#3에서는 **Dispatchers.Default**라는 파라미터를 사용했다. **해당 디스패처는 코루틴이 GlobalScope에서 실행될경우 사용되며 공통으로 사용되는 백그라운드 스레드풀을 이용**한다고 한다. (**launch(Dispatchers.Default {}  = GlobalScope.launch{}** )

**newSingleThread** → **코루틴을 실행할때마다 새로운 스레드를 만들어줌!** (비싼 리소스)  → 꼭 해제를 해줘야하는데, 다음과 같이 use 블록에서 사용해주는것이 좋음 use 블록 안에서 마지막에 close를 해주기 때문에 use블록을 사용하지 않는다면 메모리릭이 발생할 수 있음

```kotlin
newSingleThreadContext("MyOwnThread").use {
       launch(it) {
            println("newSingleThreadContext  : " + "I'm working in thread ${Thread.currentThread().name}")
        }
    }
```

> Debuggin coroutines and threads

코루틴들은 한 스레드에서 중단된 후 다른 스레드에서 재개될 수 있다. → 코루틴이 언제 어디서 수행중이였는지 알아보자 

![image](https://user-images.githubusercontent.com/53978090/128694276-fed580cc-fa44-4602-b2fe-d8c681982bf6.png)

JVM option에 **-Dkotlinx.coroutines.debug**를 추가해주면 어떤 코루틴에서 생성된건지 디버깅해서 볼 수 있다!  

```kotlin
import kotlinx.coroutines.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")
fun main() = runBlocking<Unit> {
    val a = async {
        log("I'm computing a piece of the answer")
        6
    }
    val b = async {
        log("I'm computing another piece of the answer")
        7
    }
    log("The answer is ${a.await() + b.await()}")
}
```

![image](https://user-images.githubusercontent.com/53978090/128694294-cd88220d-0add-4e0a-9bab-033e04e81714.png)

다음은 async코루틴 빌더로 2개의 코루틴을 만들어 실행시키는 예제이다. log함수를 만들어 현재 코루틴이 실행되고 있는 스레드를 출력하고 디버깅 옵션을 통해 실행되고 있는 코루틴의 이름, 번호를 알 수 있다.

→ @코루틴이름 #JVM이 생성하는 대로 매겨지는 번호 

> Jumping between threads

- WithContext(이동하려는 스레드)

    : 스레드간의 이동이 필요할때 사용! 

ex) background 스레드 → main 스레드로 이동 

```kotlin
import kotlinx.coroutines.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")
fun main() = runBlocking<Unit> {
    newSingleThreadContext("Ctx1 ").use { ctx1 ->
        newSingleThreadContext("Ctx2").use { ctx2 ->
            runBlocking(ctx1) {
                log("Started in ctx1")
                withContext(ctx2){
                    log("Working in ctx2")
                }
                log("Back to ctx1")
            }
        }
    }
}
```

![image](https://user-images.githubusercontent.com/53978090/128694305-b4dc8276-a87e-4f59-a020-ad09c1589cc5.png)

newSingleThreadContext를 사용해서 ctx1과 ctx2라는 스레드를 각각 만들었고, runBlocking을 사용해서 

내부가 실행될때까지 ctx1을 block한다. withContext()를 사용해서 ctx1이 block되는 동안 ctx2가 실행되고, ctx2가 다 끝나면 다시 ctx1로 돌아온다.

> Job in the context

[Job] : 코루틴 컨텍스트에서 Job element탐색

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    println("My job is ${coroutineContext[Job]}")

    launch {
        println("My job is ${coroutineContext[Job]}")
    }

    async {
        println("My job is ${coroutineContext[Job]}")
    }
}
```

![image](https://user-images.githubusercontent.com/53978090/128694339-9dcbab2a-9ca4-4600-a6f0-95e72fd6c20d.png)

: 실행한 코루틴의 컨텍스트에서 job이라는 요소를 꺼내봤더니, job이 있다...Dispathcers도 있다..이런걸 확인하는 용도로 사용한다고 한다 ㅎㅎ

저번에 코루틴 취소에서 사용했던 isActive는 Job이 실행되고 있는지를 확인하는  확장 프로퍼티 !

> Children of a coroutine

코루틴이 다른 코루틴 스코프 안에서 실행된다면 바깥에 있는 코루틴(부모코루틴)의 context를 상속하고 자식 코루틴이 된다. 부모 코루틴은 자식 코루틴이 끝날때까지 기다려야하고, 부모 코루틴이 취소되면 다른 자식들도 취소된다는 특징이 있다. 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {//#1
    val request = launch {//#2
        println("test!"+ "[${Thread.currentThread().name}]")

        GlobalScope.launch {//#3
            println("job1 : I run in GlobalScope and execute independently!" + "[${Thread.currentThread().name}]")
            delay(1000)
            println("job1: I am not affected by cancellation of the request"+ "[${Thread.currentThread().name}]")
        }

        launch {//#4
            delay(100)
            println("job2 : I am a child of the request coroutine"+ "[${Thread.currentThread().name}]")
            delay(1000)
            println("job2 : I will not execute this line if my parent request is cancelled"+ "[${Thread.currentThread().name}]")
        }
    }
    delay(500)
    request.cancel()
    delay(1000)
    println("main : Who has survived request cancellation?"+ "[${Thread.currentThread().name}]")
} 
```

![image](https://user-images.githubusercontent.com/53978090/128694354-64e6f62b-4e91-4fd1-846a-6b2aaac366e5.png)

#2는 #1의 자식 코루틴이고, launch의 파라미터로 아무것도 들어오지 않았기 때문에 runBlocking의 컨텍스트를 상속한다.

#3은 GlobalScope로 새로운 스코프를 만들었다. GlobalScope는 프로그램 전체에서 실행되는 코루틴이기 때문에 자식코루틴이 되지 않는다!! 

하지만 #4는 #2와 마찬가지로, #2의 자식 코루틴이 된다. 

 

request.cancel()을 호출해서 0.5초 뒤에 부모 코루틴 취소되었고, 자식 코루틴이 아닌 #3은 이와 상관없이 두번 다 출력 되었지만, 자식 코루틴인 #4는 취소되어 처음 print문만 출력되었고 두번째 print문은 출력되지 않았다. 

> Parental responsibilities

: 모든 자식 코루틴들이 실행 완료될때까지 부모 코루틴은 계속 대기해야함 → join을 하나하나 써주지 않아도 된다! 

#1은 #2가 끝날때까지 기다려주고, #2는 #3이 끝날때까지 기다려준다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {//#1
    val request = launch {//#2
        repeat(3) { i ->
            launch { //#3
                delay((i + 1) * 200L)
                println("Coroutine $i is done")
            }
        }
        println("request: I'm done and I don't explicitly join my children the")
    }
    println("Now processing of the request is complete")
}
```

![image](https://user-images.githubusercontent.com/53978090/128694387-f6c8379e-1fb0-48ed-a3d9-d5356399cc04.png)

> Combining context elements

코루틴 엘리먼트를 합쳐서 하나로 만들 수 있다. 코루틴 빌더의 매개변수로 이름이 test고 디스패처가 Default인 코루틴을 만들었다.  

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    launch(Dispatchers.Default + CoroutineName("test")) {
        println("I'm working in thread ${Thread.currentThread().name}")
    }
}
```

![image](https://user-images.githubusercontent.com/53978090/128694399-4dc0fdd5-8d72-4c82-a6ef-423d9e2d2a91.png)

test라고 이름을 정해준대로 @test#2로 나온것을 볼 수 있다.

 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    launch(Dispatchers.Default) {
        println("I'm working in thread ${Thread.currentThread().name}")
    }
}
```

![image](https://user-images.githubusercontent.com/53978090/128694418-cf312a80-d759-41d1-b1ee-fd031ea2c478.png)

이름을 정해주지 않으면 coroutine이라는 이름이 된다. 

> Coroutine Scope

액티비티에서 코루틴을 사용하고 있을때 제때 종료해주지 않으면(보통은 destroy()에서 job을 종료해준다.) back키를 눌러 해당 액티비티를 벗어나면 메모리릭이 발생할 수 있다! → job.cancel()을 하나하나 해줘야할까?

⇒ 가장 좋은 방법은 Coroutine Scope를 사용해서 job들을 넣고 cancel을 한번에 하면 된다. or lifecycle에 맞게 실행시키기 ex) viewModelScope가 아닐까! 

```kotlin
import kotlinx.coroutines.*

class Activity{
    private val mainScope = CoroutineScope(Dispatchers.Default)

    fun destroy(){
        mainScope.cancel()
    }

    fun doSomething(){
        repeat(10){ i ->
             mainScope.launch {
                 delay((i + 1)* 200L)
                 println("Coroutine $i is done")
             }
        }
    }
}

fun main() = runBlocking {
    val activity = Activity()
    activity.doSomething()
    println("Launched Coroutines")
    //delay걸어주지 않으면 Main이 바로 끝나버림 
    delay(500)
    println("Destroying activity!")
    activity.destroy()
    delay(3000)
}
```

![image](https://user-images.githubusercontent.com/53978090/128694428-1511092b-2cef-44a8-b7af-72afbb491e7c.png)

mainScope라는 코루틴 스코프를 하나 만들어놓고 그 스코프 안에서 코루틴 10개를 실행시켰고 중간에 activity.cancel()을 호출했다. 코루틴이 실행되고있던 mainScope를 cancel 했기 때문에 하나씩 cancel을 해주지 않아도 바로 취소되는것을 볼 수 있다. 

---

**참고**

<[https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html)>

<[https://www.youtube.com/watch?v=0uIrl47bSTA&list=PLbJr8hAHHCP5N6Lsot8SAnC28SoxwAU5A&index=6](https://www.youtube.com/watch?v=0uIrl47bSTA&list=PLbJr8hAHHCP5N6Lsot8SAnC28SoxwAU5A&index=6)>

<[https://myungpyo.medium.com/코루틴-공식-가이드-자세히-읽기-part-5-62e886f7862d](https://myungpyo.medium.com/%EC%BD%94%EB%A3%A8%ED%8B%B4-%EA%B3%B5%EC%8B%9D-%EA%B0%80%EC%9D%B4%EB%93%9C-%EC%9E%90%EC%84%B8%ED%9E%88-%EC%9D%BD%EA%B8%B0-part-5-62e886f7862d)>
