---
title : "Android WorkManager란? "
excerpt : "Android 백그라운드 처리 방식인 WorkManager를 알아보자"

categories:
  - Android
tags :
  - Kotlin
---

## WorkManager

: 백그라운드 작업을 위해 사용하는 라이브러리, Android Jetpack 아키텍처의 구성요소중 하나 

> 안드로이드 백그라운드 처리란?

비트맵 디코딩, 저장소 액세스, 머신러닝(ML) 모델 작업, 네트워크 요청 실행 등 오래걸리는 작업은 백그라운드 스레드에서 처리해야함!

> 백그라운드 작업의 카테고리

- **즉시**

    사용자가 어플리케이션과 상호작용하는동안 작업을 완료해야함

    → Coroutine, Forground Service

- **지연**

    대부분의 작업! 네트워크 가용성 및 배터리 잔량같은 조건을 기반으로 실행시, 약간의 편차를 허용함 

    → WorkManager 

- **정시**

    작업을 정확한 시간에 실행해야함 

    → AlarmManager 

> 안드로이드 백그라운드 작업 처리 방법

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b2cff420-0f6c-4bc5-80b5-e1a75e90cda9/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b2cff420-0f6c-4bc5-80b5-e1a75e90cda9/Untitled.png)

안드로이드에서 백그라운드 작업 처리 방법은 위와 같이 여러가지가 있다. 어떤 방법을 사용할지는..백그라운드 작업을 어느 상황에 처리해야할지, 상황에 따라 알맞는 방법을 사용하면 된다. 

1. **Exact** **Timing** : 작업이 즉시 처리되어야 하는지 
2. **Best-Effort** : 작업을 처리하려고 노력하지만 실행이 취소될 수 있고, 그에 따라 결과물이 없을수도 있다.
3. **Deferrable** : 어떠한 조건이 맞는 시점에 실행되어야함 
4. **Guaranteed** **Excution** : 반드시 작업을 처리하고 원하는 결과를 얻어야함  

> WorkManager의 등장

- AlarmManager + 브로드캐스트 리시버(기존의 방법)
- JobScheduler (기존의 AlarmManager 정확한 실행을 보장하지 않게 되었음 그래서 등장)
- Firebase JobDispatcher(상황에 따라 AlarmManager와 JobScheduler를 자동으로 선택, 하지만 구글 플레이 서비스에 제한적임)
- 2018년에 등장한것이 WorkManager

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0c984b0c-fb43-401f-899a-de5c906ce665/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0c984b0c-fb43-401f-899a-de5c906ce665/Untitled.png)

> WorkManager의 특징

- 실행이 보장됨 + 제약 조건을 가지고 실행할 수 있음
- 구글 서비스의 유무에 상관없이 동작
- 상태조회 가능
- 작업 연결처리 가능
- 제한조건이 충족되었을때 즉시 실행됨

> WorkManager의 구조

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9611eb83-c730-443f-8cc1-92de3dbee7cb/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9611eb83-c730-443f-8cc1-92de3dbee7cb/Untitled.png)

> 언제 사용하는것이 좋은가?

: 앱의 종료 여부와 상관없이 수행되어야 하는 작업

ex) 이미지를 서버에 업로드, 데이터 분석하고 데이터베이스에 저장 

> WorkManager의 구성

- **WorkManager** : 개별 WorkRequest를 대기열에 넣고 실행을 관장하는 클래스
- **Worker** : 작업을 수행하는 역할
- **WorkRequest** : Worker가 어떻게 동작해야하는지를 나타냄
    - OneTimeWorkRequest : 한번만 수행
    - PeriodicWorkRequest : 반복 수행(ex) 주기적인 작업)
        - repeat interval : Work가 실행되는 반복 주기를 설정 (기본값 15분, 그보다 짧은 시간은 허용x)
        - flex interval : Work가 실행되는 최소 가동시간 (기본값 5분, 그보다 짧은 시간은 허용x, repeat interval보다 작은 값이여야함 그렇지 않다면 repeat interval 값으로 설정됨)

            → 시작시간 = repeat interval - flex interval 

- **WorkState** : 작업의 진행상태를 나타냄 → UI갱신로직구현, 이미 실행중인 작업인지 파악
- **Constraints** : 어떤 제약 조건에서 실행할지를 나타냄  ex) 기기가 충전중일때, 기기의 배터리가 부족하지 않을때, 기기의 저장공간이 부족하지 않을때

> 기본 사용 방법

1. Worker 클래스 생성

    ```kotlin
    class SimpleWorker : Worker() {
      override fun doWork(): WorkerResult {
        // 처리해야할 작업 명시 
    		//return은 SUCCESS, FAILURE, RETRY가 있다. 
        return WorkerResult.SUCCESS
      }
    }
    ```

    Worker 클래스를 상속받아야함, 한개의 doWork 추상메소드가 존재

2-1. SimpleWorker의 작업을 한번만 수행하는 코드 작성

```kotlin
val workRequest = OneTimeWorkRequestBuilder<SimpleWorker>().build()

val workManager = WorkManager.getInstance()
        
workManager?.enqueue(workRequest)
```

2-2. PeriodicWorkRequest을 사용한 여러번 반복하는 코드

PeriodicWorkRequestBuilder(반복될 인터벌 값, 해당 인터벌의 시간타입) 

```kotlin
//15분마다 작업 실행 
val workRequest = PeriodicWorkRequestBuilder<SimpleWorker>(15, TimeUnit.MINUTES).build()

val workManager = WorkManager.getInstance()

workManager?.enqueue(workRequest)
```

> 특정 제약조건을 충족해야 실행되려면?

제약조건을 나타내려면 Constraint를 사용하면 된다!

```kotlin
//배터리 충전중일때 실행 
val constraints = Constraints.Builder()
                .setRequiredNetworkType(NetworkType.CONNECTED)
								.build()

val requestConstraint  = OneTimeWorkRequestBuilder<SimpleWorker>()
                .setConstraints(constraints)
                .build()

val workManager = WorkManager.getInstance()
workManager?.enqueue(requestConstraint)                
```

> 연결된 작업을 실행하려면?

A작업을 실행한 후 B작업을 실행해야한다면 !

beginWith와 then을 사용해서 다음과 나타낼 수 있다. 

```kotlin
val workA = OneTimeWorkRequestBuilder<AWorker>().build()
val workB = OneTimeWorkRequestBuilder<BWorker>().build()

WorkManager.getInstance()?.apply{
		beginWith(workA).then(workB).enqueue() 
}
```

---

[https://developer.android.com/guide/background?hl=ko](https://developer.android.com/guide/background?hl=ko)

[https://medium.com/@limgyumin/새로운-안드로이드-백그라운드-작업-처리법-workmanager-f625e07b384c](https://medium.com/@limgyumin/%EC%83%88%EB%A1%9C%EC%9A%B4-%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%EB%B0%B1%EA%B7%B8%EB%9D%BC%EC%9A%B4%EB%93%9C-%EC%9E%91%EC%97%85-%EC%B2%98%EB%A6%AC%EB%B2%95-workmanager-f625e07b384c)

[https://medium.com/@limgyumin/workmanager-쉽게-쓰기-b71917ea8c1](https://medium.com/@limgyumin/workmanager-%EC%89%BD%EA%B2%8C-%EC%93%B0%EA%B8%B0-b71917ea8c1)


