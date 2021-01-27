---
title : "Aandroid AAC - LiveData,ViewModel예제 (java) "
excerpt : "Google CodeLab 예제 공부"

categories:
  - Android
tags :
  - Java
---
  
### LiveData 와 ViewModel

작년 9월부터 AAC(Android Architecture Components)에 발을 들이게 됐다. 

블로그만 보고 하기엔 너무 어려워서 힘들었는데 [오준석 생존코딩 모던 안드로이드](https://www.youtube.com/c/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C%EC%83%9D%EC%A1%B4%EC%BD%94%EB%94%A9/videos) 이걸 보고 어떻게 작동하는지 빠르게 훑어볼 수 있었다. (거의 맛보기이므로 추가적인 공부 필수 ㅎ)

실습은 좀 해봤지만 제대로 알고 쓰는게 아닌거 같아 Codelab 따라하면서 다시 공부하기로! 👊🏻👊🏻

---



### LiveData

: 수명주기를 인식하고, 데이터를 관찰할 때 사용됨  

활성 상태일때만 업데이트를 보냄 (= 다른 앱으로 이동하면 활성 상태 x)

 -> 사용하는 이유 

 - UI와 데이터 상태의 일치 보장
 - 메모리 누수 x
 - 최신 데이터 유지
 - 수명주기 수동으로 처리하지 않음 등등 .. 

### LifecycleOwner

: LifecycleOwner 인터페이스며, 해당 인터페이스를 구현하는 개체에 다른 구성 요소를 등록해서 변경 사항을 관찰할 수 있음

### ViewModel

: 클래스의 수명 주기를 고려하여 UI 관련 데이터를 저장하고 관리함, 구성 변경시 데이터 유지 가능 

Ex) 애뮬레이터 회전시, 컨트롤러에 저장된 모든 일시적인 UI관련 데이터들이 삭제되기 때문에 초기화가 된다. 하지만 ViewModel 을 사용하면 유지할 수 있음 

❗️여기엔 Context나 View에 관한 내용은 쓰지 않음 -> 메모리 누수가 발생할 수 있기 때문에 Ex) Context를 참조하는 Dialog나 Toast, Intent간의 이동같은거 -> View에서 처리해야함 ❗️



![활동 상태 변경에 따라 ViewModel의 수명 주기를 설명합니다.](https://developer.android.com/images/topic/libraries/architecture/viewmodel-lifecycle.png)



장치가 회전될때 Activity는 onDestroy를 호출하면서 파괴되지만, ViewModel 인스턴스는 파괴되지 않는다. 



1.  에뮬레이터 회전해도 값이 초기화 되지 않는 예제

   

   ![ezgif com-gif-maker](https://user-images.githubusercontent.com/53978090/105740092-2d911b80-5f7c-11eb-90b6-5e8317859de3.gif)



**Activity.java**

~~~
/*
 * Copyright 2019, The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.example.android.lifecycles.step3;

import androidx.lifecycle.Observer;
import androidx.lifecycle.ViewModelProvider;
import android.os.Bundle;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;
import android.util.Log;
import android.widget.TextView;

import com.example.android.codelabs.lifecycle.R;


public class ChronoActivity3 extends AppCompatActivity {

    //뷰모델 변수 선언  
    private LiveDataTimerViewModel mLiveDataTimerViewModel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.chrono_activity_3);

        //ViewModelProvide를 통해 LiveDataTimerViewModel 객체를 생성 
        mLiveDataTimerViewModel = new ViewModelProvider(this).get(LiveDataTimerViewModel.class);

        subscribe();
    }

    private void subscribe() {
        //Long 타입 라이데이터를 관찰하는 Observer
        //elapseTime을 계속 관찰하면서 textView에 set해줌
        final Observer<Long> elapsedTimeObserver = new Observer<Long>() {
            @Override
            public void onChanged(@Nullable final Long aLong) {
                String newText = ChronoActivity3.this.getResources().getString(
                        R.string.seconds, aLong);
                ((TextView) findViewById(R.id.timer_textview)).setText(newText);
                Log.d("ChronoActivity3", "Updating timer");
            }
        };

        //ViewModel의 elapsedTime을 관찰하겠다는 뜻
        mLiveDataTimerViewModel.getElapsedTime().observe(this,elapsedTimeObserver);
    }
}
~~~



**ViewModel**

~~~
/*
 * Copyright 2019, The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.example.android.lifecycles.step3;

import androidx.lifecycle.LiveData;
import androidx.lifecycle.MutableLiveData;
import androidx.lifecycle.ViewModel;
import android.os.SystemClock;

import java.util.Timer;
import java.util.TimerTask;

/**
 * A ViewModel used for the {@link ChronoActivity3}.
 */
public class LiveDataTimerViewModel extends ViewModel {

    private static final int ONE_SECOND = 1000;

    private MutableLiveData<Long> mElapsedTime = new MutableLiveData<>();

    private long mInitialTime;
    private final Timer timer;

    public LiveDataTimerViewModel() {
        mInitialTime = SystemClock.elapsedRealtime();
        timer = new Timer();

        // Update the elapsed time every second.
        timer.scheduleAtFixedRate(new TimerTask() {
            @Override
            public void run() {
                final long newValue = (SystemClock.elapsedRealtime() - mInitialTime) / 1000;

                // setValue() cannot be called from a background thread so post to main thread.
                //LiveData변수에 Data를 정해줌
                mElapsedTime.postValue(newValue);
            }
        }, ONE_SECOND, ONE_SECOND);

    }

    @SuppressWarnings("unused")  // Will be used when step is completed
    public LiveData<Long> getElapsedTime() {
        return mElapsedTime;
    }

    //뷰모델 생명주기 끝남
    @Override
    protected void onCleared() {
        super.onCleared();
        timer.cancel();
    }
}
~~~


<img width="346" alt="스크린샷 2021-01-26 오전 1 52 47" src="https://user-images.githubusercontent.com/53978090/105740123-371a8380-5f7c-11eb-8ef9-37f685c582a4.png">



이런식으로 돌아간다.

ViewModel에 관찰할 LiveData를 쓰고 setValue()나 postValue()로 값을 넣어주면 Activity에서 viewModel.선언한liveData변수.Observer(liveDat의 type)~ 해서 실시간으로 변하는 데이터를 UI로 set 해주면 된다. 

지금은 ViewModel에서 timer로 값을 set 해줬지만, 원래 Repository에 Remote(서버통신) + Local(Room) 이런식으로 데이터를 나누어 받는다. 



*LifecycleEvent 

~~~
 @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
        void addLocationListener() {
            // Note: Use the Fused Location Provider from Google Play Services instead.
            // https://developers.google.com/android/reference/com/google/android/gms/location/FusedLocationProviderApi

            mLocationManager =
                    (LocationManager) mContext.getSystemService(Context.LOCATION_SERVICE);
            mLocationManager.requestLocationUpdates(LocationManager.GPS_PROVIDER, 0, 0, mListener);
            Log.d("BoundLocationMgr", "Listener added");

            // Force an update with the last location, if available.
            Location lastLocation = mLocationManager.getLastKnownLocation(
                    LocationManager.GPS_PROVIDER);
            if (lastLocation != null) {
                mListener.onLocationChanged(lastLocation);
            }
        }
~~~



~~~
@OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
        void removeLocationListener() {
            if (mLocationManager == null) {
                return;
            }
            mLocationManager.removeUpdates(mListener);
            mLocationManager = null;
            Log.d("BoundLocationMgr", "Listener removed");
        }
~~~



@OnLifecycleEvent 를 통해 activity에서 호출하지 않고 알아서 onResume, onPause될때마다 저 코드를 실행시킨다.



2. 2개의 프로그래스바 동일한 값으로 유지하는 예제

   

![ezgif com-gif-maker (1)](https://user-images.githubusercontent.com/53978090/105740094-2ec24880-5f7c-11eb-9786-b9ed1d548ca8.gif)



액티비티 하나에 Fragment 2개를 놓고 ViewModel 을 통해 같은 상태바의 값을 가지는 예제이다.

**.xml**

~~~
<?xml version="1.0" encoding="utf-8"?><!--
  ~ Copyright 2019, The Android Open Source Project
  ~
  ~ Licensed under the Apache License, Version 2.0 (the "License");
  ~ you may not use this file except in compliance with the License.
  ~ You may obtain a copy of the License at
  ~
  ~      http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing, software
  ~ distributed under the License is distributed on an "AS IS" BASIS,
  ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  ~ See the License for the specific language governing permissions and
  ~ limitations under the License.
  -->

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.example.android.lifecycles.step5.Activity_step5">

    <fragment
        android:id="@+id/fragment1"
        android:name="com.example.android.lifecycles.step5.Fragment_step5"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

    <fragment
        android:id="@+id/fragment2"
        android:name="com.example.android.lifecycles.step5.Fragment_step5"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />
</LinearLayout>

~~~



**Activity.java**

~~~
/*
 * Copyright 2019, The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.example.android.lifecycles.step5;


import android.os.Bundle;

import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;
import androidx.lifecycle.Observer;
import androidx.lifecycle.ViewModel;
import androidx.lifecycle.ViewModelProvider;

import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.SeekBar;
import android.widget.TextView;

import com.example.android.codelabs.lifecycle.R;
import com.example.android.lifecycles.step2.ChronometerViewModel;
import com.example.android.lifecycles.step3.ChronoActivity3;

/**
 * Shows a SeekBar that should be synced with a value in a ViewModel.
 */
public class Fragment_step5 extends Fragment {

    private SeekBar mSeekBar;

    private SeekBarViewModel mSeekBarViewModel;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        View root = inflater.inflate(R.layout.fragment_step5, container, false);
        mSeekBar = root.findViewById(R.id.seekBar);

        // viewModel 객체 생성
            mSeekBarViewModel = new ViewModelProvider(requireActivity()).get(SeekBarViewModel.class);

        subscribeSeekBar();

        return root;
    }

    private void subscribeSeekBar() {

        // Update the ViewModel when the SeekBar is changed.

        mSeekBar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                //사용자가 SeekBar로 직접 progress 조절
                if(fromUser){
                    //ViewModel의 liveData에 값을 set(UI에서 직접 값을 바꾸는거니까)
                    mSeekBarViewModel.seekbarValue.setValue(progress);
                }
            }

            @Override
            public void onStartTrackingTouch(SeekBar seekBar) { }

            @Override
            public void onStopTrackingTouch(SeekBar seekBar) { }
        });

        //LiveData를 관찰하고 화면에 set해줌
        mSeekBarViewModel.seekbarValue.observe(
                requireActivity(), new Observer<Integer>() {
                    @Override
                    public void onChanged(@Nullable Integer value) {
                        if (value != null) {
                            mSeekBar.setProgress(value);
                        }
                    }
                });
    }
}

~~~



**ViewModel**

~~~
/*
 * Copyright 2019, The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.example.android.lifecycles.step5;

import androidx.lifecycle.MutableLiveData;
import androidx.lifecycle.ViewModel;

/**
 * A ViewModel used in step 5.
 */
public class SeekBarViewModel extends ViewModel {

    //두 프레그먼트가 공유할 프로그래스바 값 
    public MutableLiveData<Integer> seekbarValue = new MutableLiveData<>();
}

~~~



---

**참고**

https://developer.android.com/codelabs/android-lifecycles#0

https://developer.android.com/reference/android/arch/lifecycle/LiveData.html

https://developer.android.com/topic/libraries/architecture/viewmodel.html
