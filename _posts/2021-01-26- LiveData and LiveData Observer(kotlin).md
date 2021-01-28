---
title : "코드랩 LiveData and LiveData Observer(kotlin) "
excerpt : "코드랩 따라하면서 공부하기"

categories:
  - Android
tags :
  - Kotlin
---


### 코드랩 LiveData and LiveData observers

썼던 글이 날아가서 다시 쓰기.. 😂

### 정리

**LiveData**
: 수명주기를 인식하는 데이터 홀더 클래스
Observer가 STARTED 또는 RESUMED(활성 상태)일때, 데이터가 변경되면 정보를 알려줌 

**사용 이유**
- UI와 데이터 상태의 일치 보장
- 메모리 누출 없음 -> Observer는 수명주기 끝나면 자동 삭제
- 최신 데이터 유지
- 수명주기 자동 관리 
등이 있다고 한다. 

**LiveData 캡슐화**
: 일부 필드에 대한 직접 엑세스를 제한하기 위해 사용한다. 
~~~
priavate val _value = MutableLiveData<자료형>()
val value : LiveData<자료형>
get() = _value 
~~~
- _value는 viewModel안에서 값을 변경할때 사용하고
ex) _value.value = ~~ 
- value 는 View에서 observe 할때 사용한다.
ex) viewModel.value.observe(viewLifecycleOwner, Observer){
//action 
} 
<img width="367" alt="스크린샷 2021-01-28 오후 4 34 06" src="https://user-images.githubusercontent.com/53978090/106105165-e2972400-6186-11eb-9d8a-d0aa86a798ca.png">
--- 

### 코드
< GameFragment.kt>
~~~
/*  
 * Copyright (C) 2019 Google Inc. * * Licensed under the Apache License, Version 2.0 (the "License"); * you may not use this file except in compliance with the License. * You may obtain a copy of the License at * *     http://www.apache.org/licenses/LICENSE-2.0 * * Unless required by applicable law or agreed to in writing, software * distributed under the License is distributed on an "AS IS" BASIS, * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. * See the License for the specific language governing permissions and * limitations under the License. */  
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
import com.example.android.guesstheword.R  
import com.example.android.guesstheword.databinding.GameFragmentBinding  
  
/**  
 * Fragment where the game is played */class GameFragment : Fragment() {  
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
  
  viewModel.score.observe(viewLifecycleOwner, Observer { newScore ->  
  binding.scoreText.text = newScore.toString()  
  })  
  viewModel.word.observe(viewLifecycleOwner, Observer { newWord ->  
  binding.wordText.text = newWord  
  })  
  viewModel.eventGameFinish.observe(viewLifecycleOwner, Observer<Boolean> { hasFinished ->  
  if (hasFinished) gameFinished()  
  })  
  binding.correctButton.setOnClickListener { onCorrect() }  
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
 }  private fun updateWordText() {  
  binding.wordText.text = viewModel.word.value  
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
  action.score = viewModel.score.value?:0  
  NavHostFragment.findNavController(this).navigate(action)  
  viewModel.onGameFinishComplete()  
  
 }}
~~~

< GameViewModel.kt >
~~~
package com.example.android.guesstheword.screens.game  
  
import android.util.Log  
import androidx.lifecycle.LiveData  
import androidx.lifecycle.MutableLiveData  
import androidx.lifecycle.ViewModel  
  
class GameViewModel : ViewModel() {  
  
  // The current word  
  private val _word = MutableLiveData<String>()  
  val word: LiveData<String>  
  get() = _word  
    // The current score  
  private val _score = MutableLiveData<Int>()  
  val score: LiveData<Int>  
  get() = _score  
    // The list of words - the front of the list is the next word to guess  
  private lateinit var wordList: MutableList<String>  
  
  private val _eventGameFinish = MutableLiveData<Boolean>()  
  val eventGameFinish: LiveData<Boolean>  
  get() = _eventGameFinish  
  
  
  
    //GameFragment 들어가면 실행  
  init {  
  _word.value = ""  
  _score.value = 0  
  
 }  
  /**  
 * Resets the list of words and randomizes the order */  private fun resetList() {  
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
  
  
 /** * Moves to the next word in the list */  private fun nextWord() {  
  if (!wordList.isEmpty()) {  
  //Select and remove a word from the list  
  _word.value = wordList.removeAt(0)  
 }else{  
  onGameFinish()  
 } }  
  /** Methods for updating the UI **/  
  
  fun onSkip() {  
  _score.value = (score.value)?.minus(1)  
  nextWord()  
 }  
  fun onCorrect() {  
  _score.value = (score.value)?.plus(1)  
  nextWord()  
 }  
  //GameFragment 벗어났을때 실행 (다른화면으로 넘어감)  
  override fun onCleared() {  
  super.onCleared()  
  Log.i("GameViewModel", "GameViewModel destroyed!")  
  
 }  
  fun onGameFinish() {  
  _eventGameFinish.value = true  
  }  
  
  fun onGameFinishComplete() {  
  _eventGameFinish.value = false  
  }  
}
~~~

< ScoreFragment.kt >
~~~
/*  
 * Copyright (C) 2019 Google Inc. * * Licensed under the Apache License, Version 2.0 (the "License"); * you may not use this file except in compliance with the License. * You may obtain a copy of the License at * *     http://www.apache.org/licenses/LICENSE-2.0 * * Unless required by applicable law or agreed to in writing, software * distributed under the License is distributed on an "AS IS" BASIS, * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. * See the License for the specific language governing permissions and * limitations under the License. */  
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
 * Fragment where the final score is shown, after the game is over */class ScoreFragment : Fragment() {  
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
  viewModel.score.observe(viewLifecycleOwner, Observer { newScore ->  
  binding.scoreText.text = newScore.toString()  
  })  
  
  viewModel.eventPlayAgain.observe(viewLifecycleOwner, Observer { playAgain ->  
  if (playAgain) {  
  findNavController().navigate(ScoreFragmentDirections.actionRestart())  
  viewModel.onPlayAgainComplete()  
 }  })  
  return binding.root  
  
  }  
  
  
}
~~~

< ScoreViewModel >
~~~
package com.example.android.guesstheword.screens.score  
  
import android.util.Log  
import androidx.lifecycle.LiveData  
import androidx.lifecycle.MutableLiveData  
import androidx.lifecycle.ViewModel  
  
class ScoreViewModel(finalScore :Int) : ViewModel() {  
  private val _score = MutableLiveData<Int>()  
  val score: LiveData<Int>  
  get() = _score  
  
    private val _eventPlayAgain = MutableLiveData<Boolean>()  
  val eventPlayAgain: LiveData<Boolean>  
  get() = _eventPlayAgain  
  
    init {  
  _score.value= finalScore  
  }  
  
  fun onPlayAgain() {  
  _eventPlayAgain.value = true  
  }  
  fun onPlayAgainComplete() {  
  _eventPlayAgain.value = false  
  }  
  
}
~~~

---
**참고**

<https://developer.android.com/codelabs/kotlin-android-training-live-data#0 >

<https://developer.android.com/topic/libraries/architecture/livedata>
