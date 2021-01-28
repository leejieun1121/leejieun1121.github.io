---
title : "코드랩 ViewModel and ViewModelProvider) "
excerpt : "코드랩 따라하면서 공부 "

categories:
  - Android
tags :
  - Kotlin
---

### 코드랩 Viewmodel and ViewModelProvider 

---

![ezgif com-gif-maker (2)](https://user-images.githubusercontent.com/53978090/105855628-a1d6c800-602b-11eb-85b7-6b48cdde76fe.gif)



**GameFragment.kt**

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
import androidx.lifecycle.ViewModelProvider
import androidx.navigation.fragment.NavHostFragment
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

        binding.correctButton.setOnClickListener {  onCorrect() }
        binding.skipButton.setOnClickListener { onSkip() }
        binding.endGameButton.setOnClickListener { onEndGame() }
        return binding.root

    }

    private fun onSkip() {
        viewModel.onSkip()
        updateWordText()
        updateScoreText()
    }

    private fun onCorrect() {
        viewModel.onCorrect()
        updateWordText()
        updateScoreText()
    }
    private fun updateWordText() {
        binding.wordText.text = viewModel.word
    }

    private fun updateScoreText() {
        binding.scoreText.text = viewModel.score.toString()
    }


    private fun onEndGame(){
        gameFinished()
    }

    private fun gameFinished(){
        Toast.makeText(context,"Game has just finished", Toast.LENGTH_SHORT).show()
        val action = GameFragmentDirections.actionGameToScore()
        //여기서 ScoreFragment 의 Bundle로 값을 참조할 수 있게 점수를 넘겨준 것
        action.score = viewModel.score
        NavHostFragment.findNavController(this).navigate(action)

    }
}

~~~



**GameViewModel.kt**

~~~
package com.example.android.guesstheword.screens.game

import android.util.Log
import androidx.lifecycle.ViewModel

class GameViewModel : ViewModel() {

    // The current word
    var word = ""

    // The current score
    var score = 0

    // The list of words - the front of the list is the next word to guess
    private lateinit var wordList: MutableList<String>

    //GameFragment 들어가면 실행
    init {
        Log.i("GameViewModel", "GameViewModel created!")
        resetList()
        nextWord()

    }

    /**
     * Resets the list of words and randomizes the order
     */
    private fun resetList() {
        wordList = mutableListOf(
                "queen",
                "hospital",
                "basketball",
                "cat",
                "change",
                "snail",
                "soup",
                "calendar",
                "sad",
                "desk",
                "guitar",
                "home",
                "railway",
                "zebra",
                "jelly",
                "car",
                "crow",
                "trade",
                "bag",
                "roll",
                "bubble"
        )
        wordList.shuffle()
    }

    /** Methods for buttons presses **/


    /**
     * Moves to the next word in the list
     */
    private fun nextWord() {
        if (!wordList.isEmpty()) {
            //Select and remove a word from the list
            word = wordList.removeAt(0)
        }
    }

    /** Methods for updating the UI **/

    fun onSkip() {
        score--
        nextWord()
    }

    fun onCorrect() {
        score++
        nextWord()
    }

    //GameFragment 벗어났을때 실행 (다른화면으로 넘어감)
    override fun onCleared() {
        super.onCleared()
        Log.i("GameViewModel", "GameViewModel destroyed!")

    }

}
~~~



**ScoreFragme.kt**

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
import androidx.lifecycle.ViewModelProvider
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
        binding.scoreText.text = viewModel.score.toString()

        return binding.root

    }


}

~~~

finish 버튼을 누르면 ScoreFragment 로 넘어가서 score를 띄워줘야하는데 

기존의 GameViewModel의 score를 사용하려고 했더니 0 이 나왔다. (onCleared()되면서 값이 날아가기때문에 0인게 당연함ㅠ )

안드로이드 권장사항에도 있듯이, 다른 책임이 있는 클래스들은 분리를 해야한다.

ScoreFragment는 따로 ScoreViewModel을 만들어주어야하고, finish버튼에서 ScoreFragment로 넘겨준 score를 ScoreViewModel에서 사용하려면 따로 ViewModelFactory를 만들어주어야 한다는 아주 중요한 사실을 깨달았다😂

**ScoreViewModelFactory.kt**

~~~
package com.example.android.guesstheword.screens.score

import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import java.lang.IllegalArgumentException

class ScoreViewModelFactory(private val finalScore: Int): ViewModelProvider.Factory {
    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        if(modelClass.isAssignableFrom(ScoreViewModel::class.java)){
            return ScoreViewModel(finalScore) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
~~~

### ViewModelFactory

: ViewModel 객체를 만들때 사용

![스크린샷 2021-01-26 오후 7 25 12](https://user-images.githubusercontent.com/53978090/105855656-a9966c80-602b-11eb-9762-f0242e46e1c6.png)

화면 회전 같은 구성 변경을 거치면 Activity가 파괴되고 다시 생성되기 때문에 데이터가 손실된다.

하지만 ViewModel에 존재하는 Data는 Activity의 생명주기와 상관없이 값을 유지한다. 



**ScoreViewModel.kt**

~~~
package com.example.android.guesstheword.screens.score

import android.util.Log
import androidx.lifecycle.ViewModel

class ScoreViewModel(finalScore :Int) : ViewModel() {
    var score = finalScore
    init {
        Log.i("ScoreViewModel", "Final score is $finalScore")
    }
}
~~~

---

**참고**

viewmodel : <https://developer.android.com/codelabs/kotlin-android-training-view-model#0>

data-binding : <https://developer.android.com/codelabs/kotlin-android-training-data-binding-basics?index=..%2F..android-kotlin-fundamentals#0>

Navigation : <https://developer.android.com/codelabs/kotlin-android-training-create-and-add-fragment?index=..%2F..android-kotlin-fundamentals#0>
