---

title : "[RxJava] 안드로이드의 RxJava 활용"
excerpt : "안드로이드에서의 RxJava 활용에 대해서 알아보자. "

categories:
  - Android
tags :
  - Java
  - RxJava
  - 리액티브 프로그래밍

---

### RxAndroid란?

: RxJava에 최소한의 클래스를 추가하여 안드로이드 앱에서 리액티브 구성 요소를 쉽고 간편하게 사용하게 만드는 라이브러리 

### 리액티브 라이브러리와 API

: RxAndroid는 기본적으로 RxJava의 리액티브 라이브러리를 이용한다. 종류가 많지만 확인해두자

> 안드로이드에서 사용할 수 있는 리액티브API와 라이브러리 목록
> 

| 리액티브 API 이름 | 설명 |
| --- | --- |
| RxLifecycle | RxJava를 사용하는 안드로이드 앱용 라이프 사이클 처리 API  |
| RxBinding | 안드로이드 UI 위젯용 RxJava 바인딩 API |
| SqlBrite | SQLiteOpenHelper와 ContentResolver 클래스의 wrapper 클래스로 쿼리에 리액티브 스트림을 도입 |
| Android-ReactiveLocation | 안드로이드용 리액티브 위치 API 라이브러리(RxJava 1.x 기준) |
| RxLocation | 안드로이드용 리액티브 위치 API 라이브러리(RxJava 2.x 기준) |
| rx-preferences | 안드로이드용 리액티브 SharedPreferences 인터페이스 |
| RxFit | 안드로이드용 리액티브 Fit 라이브러리 |
| RxWear | 안드로이드용 리액티브 웨어러블 API 라이브러리 |
| RxPermissions | RxJava에서 제공하는 안드로이드 런타임 권한 라이브러리 |
| RxNotification | RxJava로 notifiaction을 관리하는 API |
| RxClipboard | 안드로이드 클립 보드용 RxJava 바인딩 API |
| RxBroadcast | 안드로이드 Broadcast 및 LocalBroadcast에 관한 RxJava 바인딩 API |
| RxAndroidBle | 블루투스 LE(Bluetooth Low Energy)장치를 다루기 위한 리액티브 라이브러리 |
| RxImagePicker | 갤러리 또는 카메라에서 이미지를 선택하기 위한 리액티브 라이브러리 |
| ReactiveNetwork | 네트워크 연결 상태나 인터넷 연결 상태를 확인하는 리액티브 라이브러리 |
| ReactiveBeacons | 주변에 있는 블루투스 LE 기반의 비콘을 수신하는 리액티브 라이브러리 |
| RxDataBinding | 안드로이드 데이터 바인딩 라이브러리용 RxJava2 바인딩 API |

### RxAndroid 기본

:RxAndroid의 기본 개념은 RxJava와 동일하다!

> RxJava의 구성요소
> 
- **Observable**: 비즈니스 로직을 이용해 데이터를 **발행**
- **Observer**: Observable에서 발행한 데이터를 **구독**
- **Scheduler**: 스케줄러를 통해서 Observable, 구독자가 **어느 스레드에서 실행될지 결정**할 수 있음

```java
Observable.create() //1. Observable 생성
	.subscribe() //2. 구독자 이용
	.subscribeOn(Schedulers.io()) //3. 스케줄러 이용
	.observeOn(AndroidSchedulers.mainThread())
```

Observable과 구독자가 연결되면 스케줄러에서 각 요소가 사용할 스레드를 결정 →

Observable이 실행되는 스레드는 subscribeOn()함수에서 설정 → 처리된 결과를 observeOn() 함수에 설정된 스레드로 보내 최종 처리 이런식으로 흘러간다. 

> RxAndroid에서 제공하는 스케줄러
> 

| 스케줄러 이름 | 설명 |
| --- | --- |
| AndroidSchedulers.mainThread() | 안드로이드의 UI 스레드에서 동작하는 스케줄러 |
| HandlerScheduler.from(handler) | 특정 핸들러에 의존하여 동작하는 스케줄러 |

mainThread()는 스케줄러 내부에서 직접 MainLooper에 바인딩하며, from()함수는 개발자가 임의의 Looper객체를 설정할 수 있다. 즉, **AndroidSchedulers.mainThread() 와 AndroidSchedulers.from(Looper.getMainLooper())는 동일 !**

### Hello world 예제

RxAndroid를 사용해서 Hello world를 출력하는 프로그램을 만들어보자.

```java
public class MainActivity extends AppCompatActivity {
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        Observer<String> observer = new DisposableObserver<String>() { //Observer 구독자 생성(수신)
            @Override
            public void onNext(@NonNull String s) { //Hello world 출력
                binding.tvMain.setText(s);
            }

            @Override
            public void onError(@NonNull Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        };

        Observable.create(new ObservableOnSubscribe<String>() { //Observable 생성 -> 데이터 발행(송신)
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> emitter) throws Throwable {
                emitter.onNext("Hello world");//Hello world 전달
                emitter.onComplete();//완료
            }
        }).subscribe(observer);//subscribe 호출시 데이터 발행(구독자 등록) 
    }
}
```

DisposableObserver는 메모리 누수를 방지하기 위한 Observer이며, Thread-Safe하다고 한다.  [https://hjlab.tistory.com/418](https://hjlab.tistory.com/418)

Observable에서 onNext로 “Hello world”라는 값을 발행하고, 구독자는 onNext로 발행된 값을 텍스트뷰에 출력하는 예제다.

또한 람다식을 이용하면 위의 콜백함수를 사용한 긴 코드를 더 간단하게 바꿀수도 있다.

```java
public class MainActivity extends AppCompatActivity {
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        Observable.<String>create(s -> {
            s.onNext("Hello world");
            s.onComplete();
        }).subscribe(o -> binding.tvMain.setText(o));
    }
}
```

그리고 just()와 메서드 래퍼런스를 사용하면 더 더 간단하게 바꿀 수 있다!!

```java
public class MainActivity extends AppCompatActivity {
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        Observable.just("Hello world")
                .subscribe(binding.tvMain::setText);
    }
}
```

```
Hello world
```

### 제어 흐름

Iterable 객체에서 apple을 찾으면 'apple'을 출력하고 그렇지 않으면 'Not found'를 출력하는 예제를 작성해보자.

```java
public class MainActivity extends AppCompatActivity {
    public static final String TAG = MainActivity.class.getSimpleName();
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        Iterable<String> samples = Arrays.asList( //Iterable 객체를 만들기
                "banana", "orange", "cherry", "mango", "pineapple", "melon", "apple"
        );

        binding.btnApple.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Observable.fromIterable(samples) //만들어둔 Iterable으로 데이터를 하나씩 발행 
                        .filter(s -> s.contains("apple")) //apple을 포함한 단어만 걸러짐 -> pineapple, apple 
                        //skipWhiles(s -> s.contains("apple"))
                        .first("Not found") //그중 가장 첫번째 데이터만 발행 
                        .subscribe(fruit -> binding.tvMain.setText(fruit)); //subscribe 호출 -> 발행된 데이터 출력 
            }
        });
    }
}
```

```
pineapple
```

먼저 과일들의 정보를 담은 samples라는 Iterable객체를 만들어주고, 버튼을 클릭하면 setOnClickListener가 호출되어 만들어둔 데이터들이 하나씩 발행된다. 그리고 filter를 통해 apple이 포함된 단어만 걸러지고 first()를 통해 걸러진 리스트 중에서 가장 첫번째 데이터를 가져와 출력한다. first()의 인자로는 "Not found"를 넣어줬는데 걸러진 리스트가 아무것도 없다면 default로 출력된다.

### RxLifecycle라이브러리

RxAndroid에는 RxLifecycle이라는 라이브러리를 제공해서 안드로이드의 액티비티와 프래그먼트의 라이프 사이클을 RxJava에서 사용할 수 있게한다. (안드로이드와 UI 라이프 사이클을 대체한다기보다 구독(subscriptions)할 때 발생할 수 있는 메모리 누수를 방지하기 위해 사용하고, 완료하지 못한 구독을 자동으로 해제(dispose)한다.) 

> **RxLifecycle 라이프 사이클 컴포넌트**
> 

| 컴포넌트 | 설명 |
| --- | --- |
| RxActivity | 액티비티에 대응 |
| RxDialogFragment | Native/Support 라이브러리인 DialogFragment에 대응 |
| RxFragment | Native/Support 라이브러리인 Fragment에 대응 |
| RxPreferenceFragment | PreferenceFragment에 대응 |
| RxAppCompatActivity | Support 라이브러리 AppCompatActivity에 대응 |
| RxAppCompatDialogFragment | Support 라이브러리 AppCompatDialogFragment에 대응 |
| RxFragmentActivity | Support 라이브러리 FragmentActivity에 대응 |

```java
public class MainActivity extends RxAppCompatActivity {
    public static final String TAG = MainActivity.class.getSimpleName();
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        Observable.interval(0, 1, TimeUnit.SECONDS)
                .map(String::valueOf)
                .compose(bindToLifecycle())
                .subscribe(s -> Log.d(TAG, s));

    }
}
```

```
2021-12-16 01:04:42.651 16406-16435/com.example.rxandroidexample D/MainActivity: 3
2021-12-16 01:04:43.651 16406-16435/com.example.rxandroidexample D/MainActivity: 4
2021-12-16 01:04:44.048 16406-16406/com.example.rxandroidexample I/Choreographer: Skipped 106 frames!  The application may be doing too much work on its main thread.
2021-12-16 01:04:44.651 16406-16435/com.example.rxandroidexample D/MainActivity: 5
2021-12-16 01:04:45.652 16406-16435/com.example.rxandroidexample D/MainActivity: 6
2021-12-16 01:04:46.650 16406-16435/com.example.rxandroidexample D/MainActivity: 7
2021-12-16 01:04:47.655 16406-16435/com.example.rxandroidexample D/MainActivity: 8
2021-12-16 01:04:48.650 16406-16435/com.example.rxandroidexample D/MainActivity: 9
```

책에서는 Butterknife 라이브러리를 이용해서 Activity가 onDestroy()됐을때 자동해제 되는 예제를 사용했는데, 굳이 이 예제를 위해 지금은 잘 사용하지 않는 Butterknife라이브러리를 추가할 필요가 없을거같아 1초 단위로   0초부터 계속 값을 발행하다가 앱이 종료되면(ex 홈키 눌러서 나가기) 구독을 해제해서 로그찍는것을 멈추는 예제를 가져왔다. 

bindToLifecycle()를 사용하기 위해 RxAppCompatActivity로 변경해주고, bindToLifecycle()을 통해 Activity의 라이프사이클에 맞춰 메모리 참조를 해제하게 된다. Activity의 라이프사이클에 따라 자동으로 해제를 해주기 때문에 주석처리하면 계속해서 로그가 찍히는 것을 확인할 수 있다. 

compose()는 **전달받은 observable을 새로운 observable로 변환**하는 역할을 하는 함수고, lamda 를 대입하여 반복되는 패턴이나 특정 로직을 함수화 해서 적용할 수 있다고 한다. [https://aroundck.tistory.com/6227](https://aroundck.tistory.com/6227) 

### UI 이벤트 처리

- **이벤트 리스너**

: 콜백 메서드 하나를 포함하는 뷰 클래스 안의 인터페이스, 리스너가 등록된 뷰 UI안의 아이템과 사용자 사이에 상호 작용이 발생할 때 안드로이드 프레임워크가 호출한다.

> **이벤트 리스너 인터페이스에 포함된 콜백 메서드**
> 

| 콜백 메서드 이름 | 설명 |
| --- | --- |
| onClick() | View.OnClickListener에서 콜백, 사용자가 아이템을 터치하거나 내비게이션 키 혹은 트랙볼로 해당 아이템을 포커스하여 enter 키 혹은 트랙볼을 눌렀을때 호출 |
| onLongClick() | View.OnFocusChangeListener에서 콜백, 사용자가 아이템을 길게 터치하거나 내비게이션 키 혹은 트랙볼로 해당 아이템을 포커스하여 enter키 혹은 트랙볼(1초 이상) 길게 눌렀을 때 호출 |
| onFocusChange() | View.OnFocusChangeListener에서 콜백, 사용자가 내비게이션 키 또는 트랙볼을 사용하여 아이템 위로 움직이게 하거나 포커스가 벗어날때 호출 |
| onKey() | View.OnKeyListener에서 콜백, 사용자가 아이템을 포커스한 후 디바이스에 있는 키를 누르거나 놓았을때 호출 |
| onTouch() | View.OnTouchListener에서 콜백, 사용자가 아이템 경계 안에서 스크린을 누르거나, 놓거나, 어떤 움직임을 포함하는 터치 이벤트 액션을 실행할 때 호출 |
| onCreateContextMenu() | View.OnCreateContectMenuListener에서 콜백, 길게 터치하거나 누른 결과로 컨텍스트 메뉴가 열렸을 때 호출 |

### onClick()의 Observable 활용 예시  1

```java
public class MainActivity extends RxAppCompatActivity {
    public static final String TAG = MainActivity.class.getSimpleName();
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        getClickEventObservable()
                .map(s -> "clicked")//전달된 view정보 -> "clicked"로 변경 
                .subscribe(getObserver());
    }

    private Observable<View> getClickEventObservable(){
        return Observable.create(new ObservableOnSubscribe<View>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<View> emitter) throws Throwable {
                binding.btnApple.setOnClickListener(emitter::onNext); //버튼 클릭 -> view정보를 onnext에 전달 
            }
        });
    }

    private DisposableObserver<? super String> getObserver(){
        return new DisposableObserver<String>() {
            @Override
            public void onNext(@NonNull String s) {
                binding.tvMain.setText(s);//전달된 "clicked" 텍스트뷰에 설정 
            }

            @Override
            public void onError(@NonNull Throwable e) {
                binding.tvMain.setText("Error");
            }

            @Override
            public void onComplete() {
                binding.tvMain.setText("Completed");
            }
        };
    }
}
```

```
clicked
```

맨처음 예제와 똑같이 데이터를 발행하는 Observable 부분과 데이터를 수신하는 DisposableObserver를 만들었다. btnApple이 눌리면 setOnClickListener가 호출되고, 클릭한 아이템이 있는 View정보를 onNext함수의 값으로 전달한다. 발행된 데이터는 map을 통해 Observable<View> → Observable<String>으로 변경되고 옵저버는 변경해준 "clicked"를 출력한다. 

*map으로 변경해주지 않으면 무슨 값이 나올까 했는데 , map을 주석처리하면 

```
com.google.android.material.button.MaterialButton{f4f5f6f VFED..C.. ...P....544,464-896,656 #7f080062 app:id/btn_apple}
```

이런값이 출력된다 ㅎㅎ

```java
public class MainActivity extends RxAppCompatActivity {
    public static final String TAG = MainActivity.class.getSimpleName();
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        Observable.create(emitter -> {
            binding.btnApple.setOnClickListener(emitter::onNext);
        }).map(s -> "clicked")
                .subscribe(text -> binding.tvMain.setText(text));
    }
}
```

방금 예제를 람다식을 사용해서 이렇게 바꾸면 아주 간단해진다! 

그리고 RxView예제도 간단하게 나와있었는데, 현재 rxbinding3에서는 RxView를 지원하지 않는다고한다!

### onClick()의 Observable 활용 예시  2

```java
public class MainActivity extends RxAppCompatActivity {
    public static final String TAG = MainActivity.class.getSimpleName();
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        Observable.<View>create(emitter -> binding.btnApple.setOnClickListener(emitter::onNext))
        .map(local -> 7)
        .flatMap(num -> {
          Random random = new Random();
          int data = random.nextInt(10);
          return Observable.just("local : " + num, "remote : "+ data,"result = "+ (num == data));
        }).subscribe(it -> Log.d(TAG,it));
    }

```

```
2021-12-16 16:25:30.840 23568-23568/com.example.rxandroidexample D/MainActivity: local : 7
2021-12-16 16:25:30.840 23568-23568/com.example.rxandroidexample D/MainActivity: remote : 0
2021-12-16 16:25:30.840 23568-23568/com.example.rxandroidexample D/MainActivity: result = false
2021-12-16 16:25:35.264 23568-23568/com.example.rxandroidexample D/MainActivity: local : 7
2021-12-16 16:25:35.264 23568-23568/com.example.rxandroidexample D/MainActivity: remote : 6
2021-12-16 16:25:35.264 23568-23568/com.example.rxandroidexample D/MainActivity: result = false
2021-12-16 16:25:36.672 23568-23568/com.example.rxandroidexample D/MainActivity: local : 7
2021-12-16 16:25:36.672 23568-23568/com.example.rxandroidexample D/MainActivity: remote : 0
2021-12-16 16:25:36.673 23568-23568/com.example.rxandroidexample D/MainActivity: result = false
2021-12-16 16:25:37.574 23568-23568/com.example.rxandroidexample D/MainActivity: local : 7
2021-12-16 16:25:37.574 23568-23568/com.example.rxandroidexample D/MainActivity: remote : 7
2021-12-16 16:25:37.575 23568-23568/com.example.rxandroidexample D/MainActivity: result = true
```

마찬가지로 버튼이 눌리면 setOnClickListener를 호출하고, 클릭한 View의 정보가 전달되어 map을 통해 7이라는 값으로 변경된다. 그리고 flatMap을 사용해서  발행된 7과 랜덤으로 발행된 data의 숫자가 일치하는지를 판단한 후 local remote result 3개의 아이템으로 결과를 출력한다.

### 액티비티 중복 실행 문제 해결

```java
public class MainActivity extends RxAppCompatActivity {
    public static final String TAG = MainActivity.class.getSimpleName();
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        Observable.create(emitter -> binding.btnApple.setOnClickListener(emitter::onNext))
                .debounce(1000, TimeUnit.MILLISECONDS)
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe( s -> {
                    SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss", Locale.KOREA);
                    String time = sdf.format(Calendar.getInstance().getTime());
                    binding.tvMain.setText("Clicked : "+ time);
                });
    }
}
```

```
Clicked : 21:42:02
Clicked : 21:42:26
```

버튼을 클릭하면 클릭한 날짜와 시간을 출력하는 예제다. 이전 예제들과는 다르게 debounce라는 메소드를 사용해서  중복클릭을 방지했다. 아무리 빠르게 많이 눌러도 인수로 넘겨준 1000ms 동안 가장 마지막에 누른 이벤트만 처리되어 데이터를 발행한다. 

> **debounce()**
> 
- debounce()원형
    
    ```java
    @CheckReturnValue
    @SchedulerSupport(SchedulerSupport.COMPUTATION)
    @NonNull
    public final Observable<T> debounce(long timeout, @NonNull TimeUnit unit) {
        return debounce(timeout, unit, Schedulers.computation());
    }
    ```
    

빠르게 연속 이벤트를 처리하는 흐름 제어 함수이며, 어떤 이벤트가 입력되고 지정한 시간동안 추가 이벤트가 발생하지 않으면 마지막 이벤트를 최종적으로 발행한다.

![image](https://user-images.githubusercontent.com/53978090/147684737-8b94c9dc-483e-4a4f-8f1d-537be0b0005f.png)

### 추천 검색어 기능 구현하기

이번엔 TextChangedListener를 이용하여 검색할 키워드를 입력하고 500ms안에 다른 문자를 입력하지 않으면 검색을 시작하는 코드를 작성해보자. 

```java
public class MainActivity extends RxAppCompatActivity {
    public static final String TAG = MainActivity.class.getSimpleName();
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        Observable.create(emitter -> binding.etSearch.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence charSequence, int i, int i1, int i2) {

            }

            @Override
            public void onTextChanged(CharSequence charSequence, int i, int i1, int i2) {
                emitter.onNext(charSequence);
            }

            @Override
            public void afterTextChanged(Editable editable) {

            }
        })).debounce(500, TimeUnit.MILLISECONDS)
                //.filter(s -> !TextUtils.isEmpty(s.toString()))
								.filter(s -> s.toString().length() >= 0)
                .observeOn(AndroidSchedulers.mainThread())//debounce가 computation 스케줄러에서 동작하기떄문에 main스레드로 text출력하기 위해 바꿔줘야함 
                .subscribe(word -> binding.tvMain.setText(word.toString()));

    }
}
```

debounce()를 사용해서 **500ms 동안 사용자가 아무리 editText에 입력을 해도 가장 마지막에 입력된 문자열만 filter에 넘어가게 되고**, 해당 문자열이 비어있지 않는다면 textview에 출력해준다. 

책의 예제에 나와있는 .filter( s → !TextUtils.isEmpty(s.toString())은 무조건 edittext에 하나라도 입력을 했을때만 textView에 반영을 하기 때문에, 다 지우면 마지막 하나가 그대로 남아있다. length() ≥ 0을 하면 지운 결과까지 잘 반영되는데 더 좋은 코드가 있는지 찾아봐야겠다. 

> onTextChanged(charSequence, int i, int i1, int i2)
> 
- charSequnece: 사용자가 새로 입력한 문자열을 포함하는 EditText객체의 문자열
- i  : 새로 추가된 문자열의 시작 위치값
- i1 : 새 문자열 대신 삭제된 기존 문자열의 수
- i2 : 새로 추가된 문자열의 수

### RecyclerView의 아이템을 클릭했을시 해당 아이템의 정보 알려주기

책에 여러가지 예제가 있었는데, 자주 쓰이는 리싸이클러뷰에 RxAndroid를 이렇게 사용하는구나 정도로만 보면 좋을것 같아 가져왔다. 

![image](https://user-images.githubusercontent.com/53978090/147684671-82064cc4-118a-4df5-91f6-0e4b27f6d939.png)

리싸이클러뷰를 만드는 법은 생략하고 RxAndroid가 쓰인 위주로 코드를 살펴보면 다음과 같다. 

```java
public class MainActivity extends AppCompatActivity {
	
	...
		private Observable<MainItem> getItemObservable() { //설치된 어플리케이션 정보를 가져와 만들어둔 MainItem으로 변경

        final PackageManager pm = this.getPackageManager();
        Intent i = new Intent(Intent.ACTION_MAIN, null);
        i.addCategory(Intent.CATEGORY_LAUNCHER);//CATEGORY_LAUNCHER타입의 앱만 가져옴 

        return Observable.fromIterable(pm.queryIntentActivities(i, 0))
                .sorted(new ResolveInfo.DisplayNameComparator(pm))//앱이름으로 정렬 
                .subscribeOn(Schedulers.io())//위치 상관없음, 앱 리스트 가져와 정렬하는거 io스케줄러에서 진행 
                .observeOn(Schedulers.io())//해당 데이터를 MainItem으로 변경하는거도 io스케줄러에서 
                .map(item -> { //이미지와 타이틀 추출 
                    Drawable image = item.activityInfo.loadIcon(pm);
                    String title = item.activityInfo.loadLabel(pm).toString();
                    return new MainItem(image, title);
                });
    }
}
```

먼저 PckageManager클래스를 이용해서 설치된 어플리케이션의 정보를 가져와 MainItem객체로 만들어주는 함수를 만든다. 

```java
public class MainActivity extends AppCompatActivity {
	
	...
		getItemObservable()
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(item ->{
                adapter.updateItems(item);
                adapter.notifyDataSetChanged();
            }
        );
}
```

 만들어둔 getItemObservable()구독해서 MainItem()데이터들을 리싸이클러뷰 리스트에 넣어준다.

 데이터를 다 넣어주고 난뒤, 클릭 이벤트를 처리해야한다. 리액티브 프로그래밍 방식으로 어댑터에서 클릭 이벤트를 처리하기 위해서는 어떤걸 사용할까?? 

```java
public class MainRecyclerAdapter extends RecyclerView.Adapter<MainRecyclerAdapter.MainViewHolder> {
		...
    private PublishSubject<MainItem> publishSubject;

		MainRecyclerAdapter() {
	        this.publishSubject = PublishSubject.create();
    }

}
```

구독자가 없더라도 실시간 처리되어 소비해야하는 Click이벤트의 특성때문에 PublisuSubject라는 뜨거운Observerble을 사용해야한다. (주로 클릭이벤트에 사용된다고 함!!)  먼저 Adapter의 생성자에서 PublishSubject객체를 만들어준다. 

```java
static class MainViewHolder extends RecyclerView.ViewHolder {
		...
		Observable<MainItem> getClickObserver(MainItem item) {
            return Observable.create(e -> binding.itemLayout.setOnClickListener(
                    view -> e.onNext(item)
            ));
    }

}
```

그리고 ViewHolder에서 getClickObserver라는 함수를 만들어 Click이벤트를 분리해준다.(bind에서 setOnClickListener를 누르면 리스너를 통해 콜백하던 방식과 다르게, 리액티브 프로그래밍에서는 Click이벤트를 분리된 Observable에 생성한다고 한다.) 

```java
public class MainRecyclerAdapter extends RecyclerView.Adapter<MainRecyclerAdapter.MainViewHolder> {
		...
    @Override
    public void onBindViewHolder(MainViewHolder holder, int position) {
				...
        holder.getClickObserver(list.get(position)).subscribe(publishSubject);
    }

}
```

클릭한 아이템마다 해당 position의 값을 넘겨야 하기때문에, adapter의 onBindViewHolder에서 만들어둔 getClickObserver함수에 해당 position아이템을 넣어주고 구독자를 등록한다. 

```java
public class MainActivity extends AppCompatActivity {
		...
		MainRecyclerAdapter adapter = new MainRecyclerAdapter();
    binding.rvMain.setAdapter(adapter);
    binding.rvMain.setLayoutManager(new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false));
    adapter.getItemPublishSubject().subscribe(s -> toast(s.title));
		...
}
```

마지막으로 adapter에 설정해뒀던 PublishSubject객체를 통해 콜백을 받아 각 position의 아이템 정보를 출력하면 된다.  

RxJava로 RecyclerView의 클릭 이벤트를 구현하는 방식이 좀 낯선데, 리스너의 역할을 publishSubject로 대신한다고 보면 될거같다..! 

- 전체코드
    - MainActivity
        
        ```java
        public class MainActivity extends AppCompatActivity {
            public static final String TAG = MainActivity.class.getSimpleName();
            private ActivityMainBinding binding;
        
            @Override
            protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                binding = ActivityMainBinding.inflate(getLayoutInflater());
                setContentView(binding.getRoot());
        
                MainRecyclerAdapter adapter = new MainRecyclerAdapter();
                binding.rvMain.setAdapter(adapter);
                binding.rvMain.setLayoutManager(new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false));
                adapter.getItemPublishSubject().subscribe(s -> toast(s.title));
        
                getItemObservable()
                        .observeOn(AndroidSchedulers.mainThread())
                        .subscribe(item -> {
                                    adapter.updateItems(item);
                                    adapter.notifyDataSetChanged();
                                }
                        );
            }
        
            private Observable<MainItem> getItemObservable() { //설치된 어플리케이션 정보를 가져와 만들어둔 MainItem으로 변경
        
                final PackageManager pm = this.getPackageManager();
                Intent i = new Intent(Intent.ACTION_MAIN, null);
                i.addCategory(Intent.CATEGORY_LAUNCHER);
        
                return Observable.fromIterable(pm.queryIntentActivities(i, 0))
                        .sorted(new ResolveInfo.DisplayNameComparator(pm))
                        .subscribeOn(Schedulers.io())
                        .observeOn(Schedulers.io())
                        .map(item -> {
                            Drawable image = item.activityInfo.loadIcon(pm);
                            String title = item.activityInfo.loadLabel(pm).toString();
                            return new MainItem(image, title);
                        });
            }
        
            private void toast(String title) {
                Toast.makeText(this, title, Toast.LENGTH_SHORT).show();
            }
        }
        ```
        
    - Adpater & ViewHolder
        
        ```java
        public class MainRecyclerAdapter extends RecyclerView.Adapter<MainRecyclerAdapter.MainViewHolder> {
            private List<MainItem> list = new ArrayList<>();
            private PublishSubject<MainItem> publishSubject;
        
            MainRecyclerAdapter() {
                this.publishSubject = PublishSubject.create();
            }
        
            public PublishSubject<MainItem> getItemPublishSubject() {
                return publishSubject;
            }
        
            public void updateItems(MainItem item) {
                list.add(item);
            }
        
            @Override
            public MainViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
                ItemMainBinding binding =
                        ItemMainBinding.inflate(LayoutInflater.from(parent.getContext()), parent, false);
                return new MainViewHolder(binding);
            }
        
            @Override
            public void onBindViewHolder(MainViewHolder holder, int position) {
                holder.bind(list.get(position));
                holder.getClickObserver(list.get(position)).subscribe(publishSubject);
            }
        
            @Override
            public int getItemCount() {
                return list.size();
            }
        
            static class MainViewHolder extends RecyclerView.ViewHolder {
                ItemMainBinding binding;
        
                public MainViewHolder(ItemMainBinding binding) {
                    super(binding.getRoot());
                    this.binding = binding;
                }
        
                public void bind(MainItem mainItem) {
                    binding.tvTitle.setText(mainItem.title);
                    binding.imgMain.setImageDrawable(mainItem.image);
                }
        
                Observable<MainItem> getClickObserver(MainItem item) {
                    return Observable.create(e -> binding.itemLayout.setOnClickListener(
                            view -> e.onNext(item)
                    ));
                }
            }
        }
        ```
        
    - MainItem
        
        ```java
        public class MainItem {
            Drawable image;
            String title;
        
            public MainItem(Drawable image, String title) {
                this.image = image;
                this.title = title;
            }
        }
        ```
        
    - item_main.xml
        
        ```java
        <?xml version="1.0" encoding="utf-8"?>
        <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:tools="http://schemas.android.com/tools"
            android:id="@+id/item_layout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="20dp"
            android:orientation="horizontal">
        
            <ImageView
                android:id="@+id/img_main"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginStart="20dp"
                tools:ignore="contentDescription" />
        
            <TextView
                android:id="@+id/tv_title"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginHorizontal="20dp" />
        </LinearLayout>
        ```
        
    - activity_main.xml
        
        ```java
        <?xml version="1.0" encoding="utf-8"?>
        <androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:app="http://schemas.android.com/apk/res-auto"
            xmlns:tools="http://schemas.android.com/tools"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            tools:context=".MainActivity">
        
            <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/rv_main"
                android:layout_width="0dp"
                android:layout_height="0dp"
                app:layout_constraintBottom_toBottomOf="parent"
                app:layout_constraintLeft_toLeftOf="parent"
                app:layout_constraintRight_toRightOf="parent"
                app:layout_constraintTop_toTopOf="parent" />
        </androidx.constraintlayout.widget.ConstraintLayout>
        ```
        

### 뷰와 뷰 그룹의 스레드 관리

: 바로 위에서 설치된 어플리케이션의 리스트를 가져와(io에서 실행) 리싸이클러뷰에 보여주는(main에서 실행) 예제를 다뤘다. 메인 스레드가 멈추는것을 방지하기 위해, 다른 스레드에서 작업을 하고 결과를 받아 메인스레드에서 처리하는 구조가 필요하다. 일반 스레드에서 작업한 결과를 어떻게 UI에 업데이트 할 수 있을까?? 안드로이드는 Looper와 handler를 사용한다! 

![image](https://user-images.githubusercontent.com/53978090/147684705-bf47e81d-d7f0-4b62-81f1-35bda674014d.png)

출처: [https://frontjang.info/entry/안드로이드의-Handler-1-배경-설명](https://frontjang.info/entry/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C%EC%9D%98-Handler-1-%EB%B0%B0%EA%B2%BD-%EC%84%A4%EB%AA%85)

`개별 스레드`는 **Handler**를 통해 Message를 Message Queue에 넣고, `UI스레드`는 **Looper**를 통해 Message Queue에서 Message를 꺼내 다른 스레드에서 보내는 데이터를 처리한다.(핸들러 인스턴스의 콜백 메소드를 실행하는것이라고 함 = handleMessage) 

이를 사용하는 대표적인 예제가 AsyncTask인데, RxAndroid를 사용하면 아주 간단하게 표현할 수 있다. (책에는 AsyncTask예제가 나와있었는데, 지금은 잘 사용하지 않아서 예제는 생략했다. AsyncTask를 알고 있다면 바로 이해될것같다!)

| AsyncTask | RxAndroid |
| --- | --- |
| execute() | subscribe() |
| doInBackground() | 리액티브 연산자와 함께 사용하는 onSubscribe() |
| onPostExecuted() | observer |

### 메모리 누수

> 메모리누수란?
> 

: 참조가 완료되었지만 할당한 메모리를 해제하지 않아서 발생하며 시스템 전체 성능에 영향을 미치므로 잘 관리해야한다.

다음은 메모리 누수가 발생하는 예제다.

```java
public class MainActivity extends AppCompatActivity {
    public static final String TAG = MainActivity.class.getSimpleName();
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

//        Observable.<String>create(s -> {
//            s.onNext("Hello world!");
//            s.onComplete();
//        }).subscribe(o -> binding.tvMain.setText(o));

        Observer<String> observer = new DisposableObserver<String>() {
            @Override
            public void onNext(@NonNull String s) {
                binding.tvMain.setText(s); //텍스트뷰 참조 
            }

            @Override
            public void onError(@NonNull Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        };
        
        Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> emitter) throws Throwable {
                emitter.onNext("Hello world!");
                emitter.onComplete();
            }
        }).subscribe(observer);
    }
}
```

Observable은 안드로이드의 컨택스트를 복사하여 유지하고, onComplete(), onError()가 호출되면 내부에서 자동으로 unsubscribe()를 호출한다.

그런데 구독자가 텍스트뷰를 참조하기 때문에 액티비티가 비정상적으로 종료되면, 텍스트 뷰가 참조하는 액티비티는 종료되어도 가비지 컬렉션의 대상이 되지 못해 메모리 누수가 발생한다.

메모리 누수를 해결하기 위한 3가지 방법을 알아보자~ 

> **1. Disposable 인터페이스를 이용하여 명시적으로 자원을 해제한다.**
> 

```java
public class MainActivity extends AppCompatActivity {
    public static final String TAG = MainActivity.class.getSimpleName();
    private ActivityMainBinding binding;
    **private Disposable disposable;**

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

				DisposableObserver<String> observer = new DisposableObserver<String>() {
            @Override
            public void onNext(@NonNull String s) {
                binding.tvMain.setText(s);
            }

            @Override
            public void onError(@NonNull Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        };

      **disposable** = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> emitter) throws Throwable {
                emitter.onNext("Hello world!");
                emitter.onComplete();
            }
        }).subscribeWith(observer);
					//.subscribe(s -> binding.tvMain.setText(s));
    }

    **@Override
    protected void onDestroy() {
        super.onDestroy();
        if (!disposable.isDisposed()) {
            disposable.dispose();
        }
    }**
}
```

onCreate()에서 subscribe()를 호출하면 onDestroy()에서 직접 dispose()를 해서 메모리 참조를 해제한다. 

이전 예제와 다르게 Observer → DisposableObserver, subscribe() → subscribeWith()으로 변경되었다. 

1. Observer와 DisposableObserver의 차이
    - `DisposableObserver`
        
        : [Disposable](https://blog.yena.io/studynote/2020/12/06/Android-RxJava(4).html)이라는 인터페이스를 구현한 Observer로 Observable로부터 데이터를 계속 받다가 더이상 데이터를 받지않아도 될 때, 효율적으로 연결을 끊을 수 있는 Observer로 메모리 누수가 발행할 수 있을때 주로 사용한다. ([https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=bootpay&logNo=221128373874](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=bootpay&logNo=221128373874))
        
2. subscribe와 subscribeWith의 차이  
- `subscribe()` : void 반환
- `subscribeWith()`: 매개변수로 넣은 소비자를 구독처리하고 리턴 !([https://www.yeh35.com/80b4acec-0117-4b3e-b583-e938b2e488cf](https://www.yeh35.com/80b4acec-0117-4b3e-b583-e938b2e488cf)) / 구독자가 아닌 소비자를 전달해서 Disposable 객체를 리턴받는 방식도 있다.

> 2. RxLifecycle 라이브러리 이용
> 

앞에 RxLifecycle라이브러리에서도 나왔던것처럼, 액티비티의 부모를 AppCompatActivity → RxAppCompatActivity로 변경하고 compose()함수를 사용해서 Rxlifecycle 라이브러리를 적용하면 된다. 

```java
public class MainActivity extends **RxAppCompatActivity** {
    public static final String TAG = MainActivity.class.getSimpleName();
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

         Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> emitter) throws Throwable {
                emitter.onNext("Hello world!");
                emitter.onComplete();
            }
        })
//                 .compose(bindToLifecycle()) //1.
                 **.compose(bindUntilEvent(ActivityEvent.DESTROY))//2.**
                 .subscribe(s -> binding.tvMain.setText(s));
    }
}
```

compose()함수를 사용해서 RxLifecycle라이브러리를 설정하는 방법은 2가지가 있다.

1. **Observable 자체를 안드로이드 액티비티에 바인딩하는 방법**
    
    bindToLifecycle()을 사용하면, onCreate() - onDestroy() / onResume() - onPause() (subscribe() - unsubscribe()함수 호출) 이렇게 쌍을 이뤄 동작한다. 
    
2. **특정 콜백 메서드에 바인딩 하는 방법** 
    
    종료되는 시점은 bindUntilEvnet()함수를 선언해서 바꿀 수 있다.
    
    - ActivityEvent
    
    ```java
    public enum ActivityEvent {
        CREATE,
        START,
        RESUME,
        PAUSE,
        STOP,
        DESTROY
    }
    ```
    

위의 코드에서는 ActivityEvent.DESTROY로 설정했기 때문에 앞에 예제와 마찬가지고 View가 DESTROY됐을때 메모리 참조를 해제한다.

> **3.CompositeDisposable 클래스 이용**
> 

마지막으로 CompositeDisposable클래스를 이용하면 생성된 모든 Observable을 라이플사이클에 맞춰 한 번에 모두 해제할 수 있다. 1번 방식과 거의 똑같지만 compositeDisposable에 만들어둔 disposable을 add해주면 한번에 다 해제가 가능하기 때문에, 여러개를 사용한다면 이걸 사용하는게 가장 베스트인것 같다 : )

```java
public class MainActivity extends AppCompatActivity {
    public static final String TAG = MainActivity.class.getSimpleName();
    private ActivityMainBinding binding;
    **private CompositeDisposable compositeDisposable = new CompositeDisposable();**
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        Disposable disposable = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> emitter) throws Throwable {
                emitter.onNext("Hello world!");
                emitter.onComplete();
            }
        }).subscribe(s -> binding.tvMain.setText(s));
        **compositeDisposable.add(disposable);**
    }

    **@Override
    protected void onDestroy() {
        super.onDestroy();
        if(!compositeDisposable.isDisposed()){
            compositeDisposable.dispose();
        }
    }**
}
```

1번 방법은 DisposableObserver를 직접 해지 한다면, 3번방식은 compositeDisposable클래스에서 일괄 처리한다.

그리고 compositeDisposable에서는 clear()라는 메소드도 존재하는데, 모두 등록된 Disposable 객체를 삭제한다는 dispose()와 같은 기능을 하지만 다음과 같은 차이점이 있다.

- `clear()`
    
    : 계속 Disposable 객체를 받을 수 있다.
    
- `dispose()`
    
    : isDisposed()함수를 true로 설정해서 새로운 Disposable 객체를 받을 수 없다!
