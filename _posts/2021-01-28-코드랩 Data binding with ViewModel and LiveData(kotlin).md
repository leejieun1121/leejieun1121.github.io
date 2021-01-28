---
title : "코드랩 Data binding with ViewModel and LiveData(kotlin) "
excerpt : "코드랩 따라하면서 공부하기~"

categories:
  - Android
tags :
  - Kotlin
---


### 코드랩 Data binding with ViewModel and LiveData



현재 앱의 아키텍쳐 

![스크린샷 2021-01-28 오전 12 04 31](https://user-images.githubusercontent.com/53978090/106027521-4f220c80-610e-11eb-9c1c-1b0016c390d7.png)

ex) GameFragment의 Finish버튼은 game_fragment.xml에 정의되어 있다.

Finish버튼을 클릭하면 setOnClickListener에 의해서 , ViewModel에 있는 score LiveData를 읽어와 ScoreFragment 로 넘겨줌



변경 될 앱의 아키텍쳐 ! 즉 Data binding 을 사용하면 listener 없이 바로 통신 

= viewModel객체는 UI데이터를 보유함 !

![스크린샷 2021-01-28 오전 12 09 00](https://user-images.githubusercontent.com/53978090/106027539-53e6c080-610e-11eb-806c-05f3ace43716.png)



-> viewModel을 layout에 넣는것 ! 

View에서 binding.viewModel = viewModel 해줘야 함

리스너 : android:onClick = @{() -> viewModel.onClick()}



* Data binding 오류 파악

  한창 AAC 시작했을때 데이터바인딩을 주먹구구식으로 썼었는데, 에러가 나는 이유를 명확하게 안알려줘서 너무 답답했었다. 

  데이터바인딩 후 에러가 난다면 다음을 확인해보자

  1. Build창의 .로 끝나는 위치가 표시되는가 
  2. onClick 데이터 바인딩을 람다식으로 잘 썼는가
  3. 철자가 틀린것이 있는가 

---

< game_fragment.xml >

~~~
<?xml version="1.0" encoding="utf-8"?><!--
  ~ Copyright (C) 2019 Google Inc.
  ~
  ~ Licensed under the Apache License, Version 2.0 (the "License");
  ~ you may not use this file except in compliance with the License.
  ~ You may obtain a copy of the License at
  ~
  ~     http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing, software
  ~ distributed under the License is distributed on an "AS IS" BASIS,
  ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  ~ See the License for the specific language governing permissions and
  ~ limitations under the License.
  -->

<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable
            name="gameViewModel"
            type="com.example.android.guesstheword.screens.game.GameViewModel" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:id="@+id/game_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".screens.game.GameFragment">

        <TextView
            android:id="@+id/word_is_text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"
            android:fontFamily="sans-serif"
            android:text="@string/word_is"
            android:textColor="@color/black_text_color"
            android:textSize="14sp"
            android:textStyle="normal"
            app:layout_constraintBottom_toTopOf="@+id/word_text"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintHorizontal_bias="0.5"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintVertical_chainStyle="packed" />

        <TextView
            android:id="@+id/word_text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:animateLayoutChanges="true"
            android:text = "@{@string/quote_format(gameViewModel.word)}"
            android:fontFamily="sans-serif"
            android:textAppearance="@style/TextAppearance.AppCompat.Headline"
            android:textColor="@color/black_text_color"
            android:textSize="34sp"
            android:textStyle="normal"
            app:layout_constraintBottom_toTopOf="@+id/score_text"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintHorizontal_bias="0.5"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/word_is_text"
            app:layout_constraintVertical_chainStyle="packed"
            tools:text="&quot;Tuna&quot;" />

        <TextView
            android:id="@+id/timer_text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginStart="8dp"
            android:layout_marginEnd="8dp"
            android:layout_marginBottom="8dp"
            android:fontFamily="sans-serif"
            android:textColor="@color/grey_text_color"
            android:textSize="14sp"
            android:textStyle="normal"
            app:layout_constraintBottom_toTopOf="@+id/score_text"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            tools:text="0:00" />

        <TextView
            android:id="@+id/score_text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginStart="8dp"
            android:layout_marginEnd="8dp"
            android:fontFamily="sans-serif"
            android:textColor="@color/grey_text_color"
            android:textSize="14sp"
            android:textStyle="normal"
            app:layout_constraintBottom_toTopOf="@+id/guideline"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            android:text="@{@string/score_format(gameViewModel.score)}"
            />

        <Button
            android:id="@+id/skip_button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginStart="16dp"
            android:text="@string/skip"
            android:onClick="@{()->gameViewModel.onSkip()}"
            android:theme="@style/SkipButton"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintHorizontal_chainStyle="spread_inside"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="@+id/guideline" />

        <androidx.constraintlayout.widget.Guideline
            android:id="@+id/guideline"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            app:layout_constraintGuide_end="96dp" />

        <Button
            android:id="@+id/correct_button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginStart="8dp"
            android:layout_marginEnd="8dp"
            android:text="@string/got_it"
            android:onClick="@{()->gameViewModel.onCorrect()}"
            android:theme="@style/GoButton"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toStartOf="@+id/end_game_button"
            app:layout_constraintStart_toEndOf="@+id/skip_button"
            app:layout_constraintTop_toTopOf="@+id/guideline" />

        <Button
            android:id="@+id/end_game_button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginEnd="16dp"
            android:text="@string/end_game"
            android:onClick="@{()->gameViewModel.onEndGame()}"
            android:theme="@style/SkipButton"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintTop_toTopOf="@+id/guideline" />

    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
~~~



< GameFragment.kt >

~~~
/*
 * Copyright (C) 2019 Google Inc.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.example.android.guesstheword.screens.game

import android.os.Bundle
import android.util.Log
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.Toast
import androidx.databinding.DataBindingUtil
import androidx.fragment.app.Fragment
import androidx.lifecycle.Observer
import androidx.lifecycle.ViewModelProvider
import androidx.navigation.fragment.NavHostFragment
import androidx.navigation.fragment.findNavController
import com.example.android.guesstheword.R
import com.example.android.guesstheword.databinding.GameFragmentBinding

/**
 * Fragment where the game is played
 */
class GameFragment : Fragment() {
    private lateinit var viewModel: GameViewModel

    //데이터 바인딩
    private lateinit var binding: GameFragmentBinding

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,
                              savedInstanceState: Bundle?): View? {

        // Inflate view and obtain an instance of the binding class
        binding = DataBindingUtil.inflate(
                inflater,
                R.layout.game_fragment,
                container,
                false
        )
        Log.i("GameFragment", "Called ViewModelProvider.get")
        viewModel = ViewModelProvider(this).get(GameViewModel::class.java)
        binding.gameViewModel = viewModel
        binding.lifecycleOwner = viewLifecycleOwner

        //이건 View에서 해야하는코드가 남아있어서 냅둠 
        viewModel.eventGameFinish.observe(viewLifecycleOwner, Observer {
            if(it)
                gameFinished()

        })
        return binding.root

    }

    private fun gameFinished(){
        Toast.makeText(context,"Game has just finished", Toast.LENGTH_SHORT).show()
        val action = GameFragmentDirections.actionGameToScore()
        //여기서 ScoreFragment 의 Bundle로 값을 참조할 수 있게 점수를 넘겨준 것
        action.score = viewModel.score.value?:0
        NavHostFragment.findNavController(this).navigate(action)
        //화면 회전시 토스트가 계속 뜨는걸 막기 위해
        viewModel.onEndGameFinished()


    }
}

~~~



< score_fragment.xml >

~~~
<?xml version="1.0" encoding="utf-8"?><!--
  ~ Copyright (C) 2019 Google Inc.
  ~
  ~ Licensed under the Apache License, Version 2.0 (the "License");
  ~ you may not use this file except in compliance with the License.
  ~ You may obtain a copy of the License at
  ~
  ~     http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing, software
  ~ distributed under the License is distributed on an "AS IS" BASIS,
  ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  ~ See the License for the specific language governing permissions and
  ~ limitations under the License.
  -->

<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable
            name="scoreViewModel"
            type="com.example.android.guesstheword.screens.score.ScoreViewModel" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:id="@+id/score_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".screens.score.ScoreFragment">

        <TextView
            android:id="@+id/you_scored_text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="32dp"
            android:fontFamily="sans-serif"
            android:text="@string/you_scored"
            android:textColor="@color/black_text_color"
            android:textSize="14sp"
            android:textStyle="normal"
            app:layout_constraintBottom_toTopOf="@+id/score_text"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintHorizontal_bias="0.5"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintVertical_chainStyle="packed" />

        <TextView
            android:id="@+id/score_text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:layout_marginBottom="8dp"
            android:fontFamily="sans-serif"
            android:textColor="@color/black_text_color"
            android:textSize="34sp"
            android:text="@{String.valueOf(scoreViewModel.score)}"
            android:textStyle="normal"
            app:layout_constraintBottom_toTopOf="@+id/play_again_button"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintHorizontal_bias="0.5"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/you_scored_text"
            app:layout_constraintVertical_chainStyle="packed"
            tools:text="40" />

        <Button
            android:id="@+id/play_again_button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginBottom="60dp"
            android:text="@string/play_again"
            android:onClick="@{()-> scoreViewModel.onPlayAgain()}"
            android:theme="@style/GoButton"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent" />

    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
~~~



< ScoreFragment.kt >

~~~
/*
 * Copyright (C) 2019 Google Inc.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.example.android.guesstheword.screens.score

import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.databinding.DataBindingUtil
import androidx.fragment.app.Fragment
import androidx.lifecycle.Observer
import androidx.lifecycle.ViewModelProvider
import androidx.navigation.fragment.findNavController
import com.example.android.guesstheword.screens.game.GameViewModel
import com.example.android.guesstheword.R
import com.example.android.guesstheword.databinding.ScoreFragmentBinding

/**
 * Fragment where the final score is shown, after the game is over
 */
class ScoreFragment : Fragment() {
    private lateinit var viewModel: ScoreViewModel
    private lateinit var viewModelFactory : ScoreViewModelFactory

    override fun onCreateView(
            inflater: LayoutInflater,
            container: ViewGroup?,
            savedInstanceState: Bundle?
    ): View? {

        // Inflate view and obtain an instance of the binding class.
        val binding: ScoreFragmentBinding = DataBindingUtil.inflate(
                inflater,
                R.layout.score_fragment,
                container,
                false
        )
        //값을 넘겨받기 위해서 viewModelFactory를 따로 만들어준듯
        //GameFragment에서 finish 버튼 누를때 넘겨준 값을 ScoreViewModelFactory 인수로 넘김
        viewModelFactory= ScoreViewModelFactory(ScoreFragmentArgs.fromBundle(requireArguments()).score)
        viewModel = ViewModelProvider(this,viewModelFactory).get(ScoreViewModel::class.java)
        binding.scoreViewModel = viewModel
        binding.lifecycleOwner = viewLifecycleOwner

        viewModel.eventPlayAgain.observe(viewLifecycleOwner, Observer {
            if(it){
                findNavController().navigate(ScoreFragmentDirections.actionRestart())
                viewModel.onPlayAgainComplete()
            }
        })

        return binding.root

    }


}

~~~



### 정리

![KakaoTalk_Photo_2021-01-28-01-56-17](https://user-images.githubusercontent.com/53978090/106026167-d2426300-610c-11eb-9515-f14886d46233.jpeg)

![KakaoTalk_Photo_2021-01-28-01-56-12](https://user-images.githubusercontent.com/53978090/106026161-cf477280-610c-11eb-8748-6d795e54bd49.jpeg)



확실히 코드가 짧아진다 ㅎㅡㅎ



---

**참고**

https://developer.android.com/codelabs/kotlin-android-training-live-data-data-binding?index=..%2F..android-kotlin-fundamentals#0

