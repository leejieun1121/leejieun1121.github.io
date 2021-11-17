---

title : "[RxJava] Observable 처음 만들기"
excerpt : "Observable에 대해 알아보자"

categories:
  - Android
tags :
  - Java
  - RxJava
  - 리액티브 프로그래밍
---

### Observable 클래스의 종류

: RxJava 1.x에서는 Observable 밖에 없었지만 RxJava2.x 에서는 Observable 이 기능에 따라 세분화됨 

- `Observable`
    
    : 데이터 흐름에 맞게 알림을 보내 구독자가 데이터를 처리할 수 있도록 함 
    
- `Maybe`
    
    : reduce() 함수나 firstElement() 함수와 같이 데이터가 발행될 수 있거나 혹은 발행되지 않고도 완료되는 경우 
    
- `Flowable`
    
    : Observable에서 데이터가 발행되는 속도가 구독자가 처리하는 속도보다 현저하게 빠른 경우 발생하는 배압 이슈에 대응하는 기능 추가 제공 
    

### Observable

: 옵저버 패턴을 구현함 

> `옵저버 패턴`
> 

: **객체의 상태 변화를 관찰하는 관찰자(옵저버) 목록을 객체에 등록**하고, 상**태 변화가 있을 때마다 메서드를 호출하여 객체가 직접 목록의 각 옵저버에게 변화를 알려주는 패턴** (라이프 사이클은 존재하지 않으며, 보통 단일 함수를 통해 변화만 알림)

Observable 클래스는 다음과 같은 세 가지의 알림을 구독자에게 전달한다.

- `onNext`
    
    : Observable이 데이터의 발행을 알림 → 기존의 옵저버 패턴과 같음 
    
- `onComplete`
    
    : 모든 데이터의 발행이 완료됐음을 알림 → onComplete 이벤트는 단 한번만 발생하며, 발생한 이후에는 더 이상 onNext 이벤트가 발생하면 안됨
    
- `onError`
    
    : Observable 에서 어떤 이유로 에러가 발생 → 후에 onNext 및 onComplte 이벤트가 발생하지 않음 (즉, Observable의 실행을 종료)
    

> Observable의 팩토리 함수 구분
> 

| 팩토리함수 | 함수 |
| --- | --- |
| RxJava 1.x 기본 팩토리 함수 | create(), just(), from() |
| RxJava 2.x 추가 팩토리 함수 (from()함수 세분화) | fromArray(), fromIterable(), fromCallable(), fromFuture(), fromPublisher() |
| 기타 팩토리 함수 | interval(), range(), timer(), defer() 등  |

> just()
> 

: 인자로 넣은 데이터를 차례로 발행하기 위해 Observable을 생성 (실제 데이터의 발행은 subscribe() 함수를 호출해야 시작!)

한 개의 값을 넣을 수도 있고, 인자로 여러 개의 값을 넣을수도 있음 (단, 타입은 모두 같아야함) 

```java
public void emit() {
        Observable.just(1,2,3,4,5)
              .subscribe(System.out::println);
}
```

```
1
2
3
4
5
```

### subscribe() 함수와 Disposable 객체

: 동작시키기 원하는 것을 사전에 정의해둔 다음, 실제 그것이 실행되는 시점을 조절하기 위해 사용 

→ just() 같은 팩토리 함수로 데이터 흐름을 정의한 후 subscibe() 함수를 호출해서 실제 데이터를 발행 

- **Disposable subscribe()**
    
    : onNext와 onComplete 이벤트를 무시하고 `onError` 이벤트가 발생했을 때만 OnErrorNotImplementedException을 던짐 
    
- **Disposable subscribe(Consumer<? super T> onNext)**
    
    : `onNext` 이벤트를 처리함, onError 이벤트가 발생하면 OnErrorNotImplementedException을 던짐 
    
- **Disposable subscribe(Consumer<? super T> onNext, Consumer<? super java.lang.Throwable> onError)**
    
    : `onNext, onError` 처리 
    
- **Disposable subscribe(Consumer<? super T> onNext, Consumer<? super java.lang.Throwable> onError, Action onCOmplete)**
    
    : `onNext, onError, onComplete 이벤트 모두 처리` 
    

앞 함수 원형은 모두 Disposable 인터페이스 객체를 리턴 

> Disposable
> 

: RxJava 1.x의 구독 객체에 해당 

- **void dispose()**
    
    : Observable에게 더 이상 데이터를 발행하지 않도록 구독을 해지하는 함수 (Observable이 onComplete 알림을 보냈을 때 자동으로 dispose()를 호출해 구독을 해지함) → onComplte 이벤트가 정상적으로 발생했다면 dispose()호출할 필요 없다.
    
- **boolean isDisposed()**
    
    : Observable이 데이터를 발행하지 않는지(구독을 해지했는지) 확인하는 함수 
    

```java
public class FirstExample {
    public static void main(String args[]) {
        Observable<String> source = Observable.just("RED", "YELLOW", "GREEN");

        Disposable d = source.subscribe(
                v -> System.out.println("onNext() : value: "+ v),
                err -> System.out.println("onError() : err: "+ err.getMessage()),
                () -> System.out.println("onComplete()")
        );

        System.out.println("isDisposed() : "+d.isDisposed());
    }
}
```

```
onNext() : value: RED
onNext() : value: YELLOW
onNext() : value: GREEN
onComplete()
isDisposed() : true
```

Disposable 객체를 반환하는 Observable의 subscribe 함수를 사용했고, 세가지 인자를 넣어 onNext, onError, onComplete 이벤트를 모두 처리했다. RED → YELLOW → GREEN 순서대로 전부 출력한 뒤, onComplete를 호출하고 isDisposed()함수를 통해 구독을 잘 해지한것을 확인할 수 있다. 

### create()

: just()함수는 데이터를 인자로 넣으면 자동으로 알림 이벤트가 발생하는 반면, create() 함수는 onNext, onComplete, onError 같은 알림을 직접 호출해야함  

```java
public class FirstExample {
    public static void main(String args[]) {
        Observable<Integer> source = Observable.create(
                (ObservableEmitter<Integer> emitter) -> {
                    emitter.onNext(100);
                    emitter.onNext(200);
                    emitter.onNext(300);
                    emitter.onComplete();
                });
        source.subscribe(System.out::println); //메서드 레퍼런스 
//      source.subscribe(data -> System.out.println(data)); // 람다 표현식 
    }
}
```

```
100
200
300
```

직접 ObservableEmitter 를 만들어서 onNext에 값을 입력해주고 onComplete를 언제할건지까지 정해준뒤 마지막에 subscribe를 통해 값이 발행된다. 

> **create() 사용시 주의점**
> 

: create()를 사용하지 않고 다른 팩토리 함수를 활용하면 같은 효과를 낼 수 있기 때문에, RxJava의 javadoc에 따르면 create()는 RxJava에 익숙한 사용자만 활용하도록 권고한다고 한다.

그래도 사용해야한다면 다음과 같은 사항들을 확인하자

1. Observable이 구독 해지되었을 때 등록된 콜백을 모두 해제해야함 그렇지 않으면 잠재적으로 메모리 누수가 발생 
2. 구독자가 구독하는 동안에만 onNext와 onComplete를 호출해야함
3. 에러가 발생했을 때는 오직 onError 이벤트로만 에러를 전달해야함
4. 배압을 직접 처리해야한다.

### fromArray()

: just()와 create()는 단일 데이터를 다뤘는데, 단일 데이터가 아닐 때는 fromXXX() 계열 함수를 사용해야 한다.

→ 배열에 들어 있는 데이터를 처리할 때 사용 ! 

```java
public class FirstExample {
    public static void main(String args[]) {
        Integer[] arr = {10,20,30};
        Observable<Integer> source = Observable.fromArray(arr);
        source.subscribe(System.out::println);
    }
}
```

```
10
20
30
```

→ 사용자 정의 클래스 객체도 넣을 수 있다. 하지만 int 같은 명시적 래퍼 타입은 제대로된 결과가 나오지 않으니 toIntegerArray(배열) 등을 통해 변환해줘야한다!! 

### fromIterable()

: Observable 을 만드는 다른 방법은 Iterable 인터페이스를 구현한 클래스에서 Observable객체를 생성하는 것이다. 

Iterator 인터페이스는 이터레이터 패턴을 구현한 것으로 다음에 어떤 데이터가 있는지와 그 값을 얻어오는 것만 관여할 뿐 특정 데이터 타입에 의존하지 않는다. 

ex) ArrayList(List 인터페이스), ArrayBlockingQueue(BlockingQueue 인터페이스), HashSet(Set 인터페이스), LinkedList, Stack, TreeSet, Vector 등 

```java
public class FirstExample {
    public static void main(String args[]) {
	      List<String> names = new ArrayList<>();
	      names.add("Jerry");
        names.add("William");
        names.add("Bob");
        Observable<String> source = Observable.fromIterable(names);
        source.subscribe(System.out::println);
    }
}
```

```
Jerry
William
Bob
```

### fromCallable()

: 앞서 살펴본 from~ 함수들은 기본적인 자료구조로 Observable 을 생성하는 부분을 생성했었지만, 이번에는 기존 자바에서 제공하는 비동기 클래스나 인터페이스 연동을 다룬다.

```java
public class FirstExample {
    public static void main(String args[]) {
        Callable<String> callable = () -> {
            Thread.sleep(1000);
            return "Hello Callable";
        };

        Observable<String> source = Observable.fromCallable(callable);
        source.subscribe(System.out::println);
    }
}
```

```
Hello Callable
```

Callable 인터페이스를 구현해서 비동기 실행 후 결과를 반환하는 call() 메서드를 정의했다. 

Callable 인터페이스는 run() 메서드가 있는 Runnable 인터페이스 처럼 메서드가 하나고, 인자가 없다는 점에서 비슷하지만 실행 결과를 리턴한다. 

1초 후 Hello Callable 을 출력하는 걸 볼 수 있다. 

### fromFuture()

: Future 인터페이스 역시 비동기 계산의 결과를 구할 때 사용하고, 보통 Executor 인터페이스를 구현한 클래스에 Callable 객체를 인자로 넣어 Future 객체를 반환한다. get()을 호출하면 Callable 객체에서 구현한 계산 결과가 나올때 까지 블로킹됨 

```java
public class FirstExample {
    public static void main(String args[]) {
        Future<String> future = Executors.newSingleThreadExecutor().submit(() -> {
           Thread.sleep(1000);
           return "Hello Future";
        });
        Observable<String> source = Observable.fromFuture(future);
        source.subscribe(System.out::println);
    }
}
```

```
Hello Future
```

### fromPublisher()

: Publisher 객체는 Observable.create()와 마찬가지로 onNext()와 onComplete()함수를 호출할 수 있다. 

```java
public class FirstExample {
    public static void main(String args[]) {
        Publisher<String> publisher = new Publisher<String>() {
            @Override
            public void subscribe(Subscriber<? super String> s) {
                s.onNext("Hello Observable.fromPublisher()");
                s.onComplete();
            }
        };
        Observable<String> source = Observable.fromPublisher(publisher);
        source.subscribe(System.out::println);
    }
}
```

```
Hello Observable.fromPublisher()
```

### Single 클래스

: Observable의 특수한 형태로, Observable 클래스는 데이터를 무한하게 발행할 수 있지만 Single 클래스는 오직 1개의 데이터만 발행하도록 한정함 → 보통 결과가 유일한 서버 API를 호출할 때 유용하게 사용 

onNext()와 onComplete()함수가 onSuccess() 함수로 통합됨 →  데이터 하나가 발행과 동시에 종료

⇒ onSuccess(T value) 함수와 onError() 함수로 구성됨 

### just()

: Observable과 거의 같은 방법으로 활용 할 수 있음 

```java
public class FirstExample {
    public static void main(String args[]) {
        Single<String> source = Single.just("Hello Single");
        source.subscribe(System.out::println);
    }
}
```

```
Hello Single
```

### Observable에서 Single 클래스 사용

: Single 은 Observable의 특수한 형태로 Observable에서 변환할 수 있음 

> Observable 클래스에서 Single 클래스 활용
> 
1. **기존 Observable에서 Single 객체로 변환** 

```java
public class FirstExample {
    public static void main(String args[]) {
        Observable<String> source = Observable.just("Hello Single");
        Single.fromObservable(source);
        source.subscribe(System.out::println);
    }
}
```

```
Hello Single
```

기존 Observable에서 첫 번째 값을 발행하면 onSuccess 이벤트를 호출한 후 종료 

1. **single() 함수를 호출해 Single 객체 생성하기** 

```java
public class FirstExample {
    public static void main(String args[]) {
        Observable.just("Hello Single")
                .single("default item")
                .subscribe(System.out::println);
    }
}
```

```
Hello Single
```

single () 함수는 default value 를 인자로 가지고, Observable 에서 값이 발행되지 않을 때도 인자로 넣은 기본값을 대신 발행함 

1. **first() 함수를 호출해 Single 객체 생성하기** 

```java
public class FirstExample {
    public static void main(String args[]) {
        String[] colors = {"Red", "Yellow", "Green"};
        Observable.fromArray(colors)
                .first("default value")
                .subscribe(System.out::println);
    }
}
```

```
Red
```

여러 개의 데이터를 발행할 수 있는 Observable 을 Single 객체로 변환, 하나 이상의 데이터를 발행하더라도 첫 번째 데이터 발행 후 onSuccess 이벤트가 발생 

1. **empty Observable에서 Single 객체 생성하기**

```java
public class FirstExample {
    public static void main(String args[]) {
        Observable.empty()
                .single("default value")
                .subscribe(System.out::println);
    }
}
```

```
default value
```

empty() 함수를 통해서 Single 객체를 생성, 3번째와 마찬가지로 첫번째 데이터 발행 후 onSuccess 이벤트가 발생하고 2번째와 마찬가지로 Observable에서 값이 발행되지 않을때도 기본값을 가지는 Single 객체로 변환 가능

1. **take() 함수에서 Single 객체 생성하기** 

```java
public class FirstExample {
    public static void main(String args[]) {
        Observable.just(new Animal("Dog"), new Animal("Cat"))
                .take(1)
                .single(new Animal("default value"))
                .subscribe(System.out::println);
    }
}
class Animal{
    private String name;
    Animal(String name){
        this.name = name;
    }

    @Override
    public String toString() {
        return "name = " + name ;
    }
}
```

```
name = Dog
```

### Single 클래스의 올바른 사용 방법

: Observable 에서 Single 객체를 생성할 때 just 함수 안에 값을 여러개 넣는다면 다음과 같은 에러가 발생한다. 

```java
public class FirstExample {
    public static void main(String args[]) {
        Observable.just(new Animal("Dog"), new Animal("Cat"))
//                .take(1)
                .single(new Animal("default value"))
                .subscribe(System.out::println);
    }
}
```

```
Caused by: java.lang.IllegalArgumentException: Sequence contains more than one element!
```

그러니 한 개의 값을 발행하도록 주의하자 ! 

### Maybe 클래스

: Single 클래스와 마찬가지로 최대 데이터 하나를 가질 수 있지만, 데이터 발행 없이 바로 데이터 발생을 완료 ( 0개 or 1개) 할 수도 있음 → Single 클래스에 onComplete 이벤트가 추가된 형태

Maybe 객체는 Maybe 클래스를 이용해 생성할 수 있지만, 주로 elementAt(), firstElement(), flatMapMaybe(), lastElement(), reduce(), singleElement() 등의 함수들을 사용해서 생성한다. 

### 차가운 Observable / 뜨거운 Observable

- `차가운 Observable`
    
    : Observable을 선언하고 just(), fromIterable() 함수를 호출해도 옵저버가 subscribe() 함수를 호출하여 구독하지 않으면 데이터를 발행하지 않음 (= lazy 접근법)
    
    ⇒ 구독자가 구독하면 준비된 데이터를 처음부터 발행 
    
    ex) 웹 요청, 데이터베이스 쿼리와 파일 읽기 등 사용 
    
- `뜨거운 Observable`
    
    : 구독자가 존재 여부와 관계 없이 데이터를 발행하는 Observable, 따라서 여러 구독자를 고려할 수 있지만 구독자로서는 Observable에서 발행하는 데이터를 처음부터 모두 수신할 것으로 보장할 수 없음 
    
    ⇒ 구독한 시점부터 Observable에서 발행한 값을 받음 
    
    ex) 마우스 이벤트, 키보드 이벤트, 시스템 이벤트, 센서 데이터, 주식 가격 
    
    ❗️ 주의할점 : 배압을 고려해야함
    

> `배압`
> 

: Observable에서 **데이터를 발행하는 속도와 구독자가 처리하는 속도의 차이가 클 때 발생** → Flowable에서 처리 

- 차가운 Observable → 뜨거운 Observable 객체로 변환하는 방법

: Subject 객체를 만들거나 Connectable Observable 클래스를 활용 

### Subject 클래스

: Observable의 속성과 구독자의 속성이 모두 있음 → Observable처럼 데이터를 발행할 수도 있고, 구독자처럼 발행된 데이터를 바로 처리할 수도 있음 

ex) AsyncSubject, BehaviorSubject,  PublishSubject, ReplaySubject

### AsyncSubject 클래스

: Observable에서 발행한 **마지막 데이터를 얻어올 수 있는** Subject 클래스, 완료되기 전 마지막 데이터에만 관심이 있으며 이전 데이터는 무시 ( = 완료되기 전까지는 구독자에게 데이터를 전달하지 않다가 완료됨과 동시에 여러 구독자에게 마지막 데이터를 발행하고 종료함) 

> AsyncSubject가 데이터 발행할 때 사용
> 

```java
public class FirstExample {
    public static void main(String args[]) {
        AsyncSubject<String> subject = AsyncSubject.create();
        subject.subscribe(data -> System.out.println("Subscriber #1 =>" + data));
        subject.onNext("1");
        subject.onNext("3");
        subject.subscribe(data -> System.out.println("Subscriber #2 =>" + data));
        subject.onNext("5");
        subject.onComplete();
    }
}
```

```
Subscriber #1 =>5
Subscriber #2 =>5
```

구독자1이 1과 3을 보내기 전에 구독을 하고, 구독자2가 5를 보내기전 구독을 했지만, 이전의 값들을 무시하고onComplete 바로 직전에 발행된 5를 출력하는 것을 알 수 있다. 

> AsyncSubject가 구독자로 사용
> 

```java
public class FirstExample {
    public static void main(String args[]) {
        //Float 타입 온도 데이터를 담는 Observable 생성 
        Float[] temperature = {10.1f, 13.4f, 12.5f};
        Observable<Float> source = Observable.fromArray(temperature);

        //subject 객체를 생성하고 데이터를 수신할 수 있도록 subscribe를 호출 
        AsyncSubject<Float> subject = AsyncSubject.create();
        subject.subscribe(data -> System.out.println("Subscriber #1 =>" + data));

        //만들어둔 Observable도 데이터 수신할 수 있도록 subscribe 호출 
        source.subscribe(subject);
    }
}
```

```
Subscriber #1 =>12.5
```

→ 이게 가능한 이유는 Subject 클래스가 Observable을 상속하고 동시에 Observable 인터페이스를 구현하기 때문 ! 

```java
public abstract class Subject<T> extends Observable<T> implements Observer<T>
```

> AsyncSubject클래스에서 onComplete()함수를 호출한 후에 구독할 때
> 

```java
public class FirstExample {
    public static void main(String args[]) {
        AsyncSubject<Integer> subject = AsyncSubject.create();
        subject.onNext(10);
        subject.onNext(11);
        subject.subscribe(data -> System.out.println("Subscriber #1 =>" + data));
        subject.onNext(12);
        subject.onComplete();
        subject.onNext(13);
        subject.subscribe(data -> System.out.println("Subscriber #2 =>" + data));
        subject.subscribe(data -> System.out.println("Subscriber #3 =>" + data));
    }
}
```

```
Subscriber #1 =>12
Subscriber #2 =>12
Subscriber #3 =>12
```

onComplete가 호출되고 난 뒤 다른 값을 발행하든지 말든지 onComplete()직전에 발행된 12를 리턴받는걸 알 수 있다. 

### BehaviorSubject

: BehaviorSubject는 구독을 하면 **가장 최근 값 혹은 기본값을 넘겨주는 클래스** (값을 처음 얻을 때는 초깃값을 반환하기도 함)  

```java
public class FirstExample {
    public static void main(String args[]) {
        BehaviorSubject<Integer> subject = BehaviorSubject.createDefault(6);
        subject.subscribe(data -> System.out.println("Subscriber #1 =>" + data));
        subject.onNext(1);
        subject.onNext(3);
        subject.subscribe(data -> System.out.println("Subscriber #2 =>" + data));
        subject.onNext(5);
        subject.onComplete();
    }
}
```

```
Subscriber #1 =>6
Subscriber #1 =>1
Subscriber #1 =>3
Subscriber #2 =>3
Subscriber #1 =>5
Subscriber #2 =>5
```

BehaviorSubject 클래스는 AsyncSubject 클래스와 다르게 createDefault() 함수로 생성한다. 구독자가 subscribe() 함수를 호출했을 때 그전까지 발행한 값이 없다면 기본값을 대신 발행하기 때문

맨 처음 발행값이 없기때문에 6을 반환하고, 1과 3이 발행되자 1과 3을 출력, 3이 발행한 뒤 Subscriber2가 구독을 했는데 가장 최근에 발행된 값이 3이기때문에 3을 출력한다. 마지막으로 5가 발행되니까 Subscriber 1,2가 5를 출력한다. 

### PublishSubject

: 구독자가 subscribe() 함수를 호출하면 값을 발행하기 시작, 오직 해당 시간에 발생한 데이터를 그대로 구독자에게 전달받음 (가장 기본~)

```java
public class FirstExample {
    public static void main(String args[]) {
        PublishSubject<Integer> subject = PublishSubject.create();
        subject.subscribe(data -> System.out.println("Subscriber #1 =>" + data));
        subject.onNext(1);
        subject.onNext(3);
        subject.subscribe(data -> System.out.println("Subscriber #2 =>" + data));
        subject.onNext(5);
        subject.onComplete();
    }
}
```

```
Subscriber #1 =>1
Subscriber #1 =>3
Subscriber #1 =>5
Subscriber #2 =>5
```

### ReplaySubject

: Subject 클래스의 목적은 뜨거운 Observable을 활용하는 것인데, 차가운 Observable처럼 동작

**구독자가 새로 생기면 항상 데이터의 처음부터 끝까지 발행하는 것을 보장해줌** → 모든 데이터 내용을 저장해두는 과정 중 메모리 누수가 발생할 가능성을 염두에 두고 사용할 때 주의해야함 ! 

```java
public class FirstExample {
    public static void main(String args[]) {
        ReplaySubject<Integer> subject = ReplaySubject.create();
        subject.subscribe(data -> System.out.println("Subscriber #1 =>" + data));
        subject.onNext(1);
        subject.onNext(3);
        subject.subscribe(data -> System.out.println("Subscriber #2 =>" + data));
        subject.onNext(5);
        subject.onComplete();
    }
}
```

```
Subscriber #1 =>1
Subscriber #1 =>3
Subscriber #2 =>1
Subscriber #2 =>3
Subscriber #1 =>5
Subscriber #2 =>5
```

Subscriber 1은 1과 3이 발행될때 1과 3을 리턴받음 , Subscriber2는 1과 3이 발행된 뒤에 구독을 하는데 이때 앞서 받지 못했던 1과 3을 모두 받는다. 마지막으로 5가 발행되고 Subscriber 1과 2가 5를 리턴받는다. 

> 데이터 발행자와 수신자
> 

| 데이터 발행자 | 데이터 수신자 |  |
| --- | --- | --- |
| Observable | 구독자(Subscriber) | → subscribe() 호출 |
| Single | 옵저버(Observer) | → Observable의 수신자 |
| Maybe | 소비자(Consumer) | → subscribe()호출할 때 함수형 인터페이스인 Consumer를 인자로 넘긴다. |
| Subject |  |  |
| Completable |  |  |

### ConnectableObservable 클래스

: Subject 클래스처럼 차가운 Observable을 뜨거운 Observable로 변환, Observable을 여러 구독자에게 공유할 수 있으므로 데이터 하나를 여러 구독자에게 동시에 전달할 때 사용 

subscribe()를 호출해도 아무 동작이 일어나지 않는다. → **connect() 함수는 호출한 시점부터 subscribe() 함수를 호출한 구독자에게 데이터를 발행하기 때문 !** 

```java
public class FirstExample {
    public static void main(String args[]) {
        String[] dt = {"1", "3", "5"};
        //시간과 시간의 단위 -> 100ms 단위로 0부터 데이터를 발행
        Observable<String> balls = Observable.interval(100L, TimeUnit.MILLISECONDS)
                .map(Long::intValue)
                .map(i -> dt[i])
                .take(dt.length);

        //publish(): 여러 구독자에게 데이터를 발행하기 위해 connect()함수 호출 전 데이터 발행을 유예함
        ConnectableObservable<String> source = balls.publish();
        source.subscribe(data -> System.out.println("Subscriber #1 => "+ data));
        source.subscribe(data -> System.out.println("Subscriber #2 => "+ data));
        //데이터 발행 시작
        source.connect();

        CommonUtils.sleep(250);
        source.subscribe(data -> System.out.println("Subscriber #3 => "+ data));
        CommonUtils.sleep(100);
    }
}
```

- CommonUtils
    
    ```java
    class CommonUtils{
        public static long startTime;
        public static void exampleStart(){
            startTime = System.currentTimeMillis();
        }
        public static void exampleComplete(){
            startTime = System.currentTimeMillis();
        }
        public static void sleep(int mills){
            try{
                Thread.sleep(mills);
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }
        public static String getThreadName() {
            String threadName = Thread.currentThread().getName();
            if (threadName.length() > 30) {
                threadName = threadName.substring(0, 30) + "...";
            }
            return threadName;
        }
        public static void doSomething(){
            try{
                Thread.sleep(new Random().nextInt(100));
            }catch(InterruptedException e){
                e.printStackTrace();
            }
        }
    }
    ```
    

```
Subscriber #1 => 1
Subscriber #2 => 1
Subscriber #1 => 3
Subscriber #2 => 3
Subscriber #1 => 5
Subscriber #2 => 5
Subscriber #3 => 5
```

Subscriber1 과 Subscriber2가 구독하고 난 뒤, connect()를 호출해서 이때 데이터가 발행됨 100ms 단위로 1과 3이 발행되어 있었기 때문에 각 구독자들이 1과 3을 출력, 그리고 250ms을 기다린 후 Subscriber3이 구독을 추가(이때 이미 connect()를 호출한 이후라서 바로 5라는 데이터를 받을 수 있다. 

마지막으로 100ms를 기다린 후에는 balls객체의 데이터를 모두 발행하고 세 구독자가 모두 구독 해지된다.

> 메서드 체이닝
> 

: 팩토리 함수와 연산자를 함께 사용하는 기법
