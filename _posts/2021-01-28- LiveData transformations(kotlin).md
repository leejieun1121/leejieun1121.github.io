---
title : " 코드랩 LiveData transformations(kotlin) "
excerpt : "코드랩 따라하면서 공부하기~~ "

categories:
  - Android
tags :
  - Kotlin
---

### 코드랩 LiveData transformations 
---

![ezgif com-gif-maker (4)](https://user-images.githubusercontent.com/53978090/106124766-cbfcc700-619e-11eb-91ec-e91419b5fb01.gif)
 
 
###정리

[**companion object**](https://medium.com/@lunay0ung/kotlin-object-declaration-%EA%B7%B8%EB%A6%AC%EA%B3%A0-companion-object-feat-static-d5c97c21168) 란 !
- java의 static과 유사함 객체생성 없이 바로 접근 가능하도록 ! 
- 클래스당 하나만 가질 수 있다.

[**CountDownTimer**](https://developer.android.com/reference/android/os/CountDownTimer)
~~~
CountDownTimer(millisInFuture, countDownInterval){
override fun onTick(p0: Long) {  
   //p0: 호출된 시점부터 종료될때까지 남은 시간
}  
  
override fun onFinish() {  
	//카운트 종료 
}
~~~
 
- millisInFuture 
: 얼마동안 타이머를 실행 시킬건지 
ms(1/1000초) 단위
- countDownInterval
: 줄어드는 시간 간격 설정 

*timer.cancel() 꼭 해주기 

[**Transformation.map()**](https://developer.android.com/reference/androidx/lifecycle/Transformations.html#map(androidx.lifecycle.LiveData%3CX%3E,%20androidx.arch.core.util.Function%3CX,%20Y%3E))
: 설정된 각 값을 적용시켜서 매핑된 값을 반환함 

[**formatElapsedTime**](https://developer.android.com/reference/android/text/format/DateUtils.html#formatElapsedTime(long))
: MM:SS 또는 H:MM:SS형식으로 경과 시간 형식화함 
별도의 스레드를 사용하지 않고 UI 스레드에서 실행됨

----
### 코드

< game_fragment.xml >
~~~
package com.example.android.guesstheword.screens.game  
  
import android.os.CountDownTimer  
import android.text.format.DateUtils  
import android.util.Log  
import androidx.lifecycle.LiveData  
import androidx.lifecycle.MutableLiveData  
import androidx.lifecycle.Transformations  
import androidx.lifecycle.ViewModel  
  
class GameViewModel : ViewModel() {  
  
  //LiveData 캡슐화  
  // The current word  
  private val _word = MutableLiveData<String>()  
  val word : LiveData<String>  
  get() = _word  
  
    // The current score  
  private val _score = MutableLiveData<Int>()  
  val score : LiveData<Int>  
  get() = _score  
  
    private val _eventGameFinish = MutableLiveData<Boolean>()  
  val eventGameFinish : LiveData<Boolean>  
  get() = _eventGameFinish  
  
    // The list of words - the front of the list is the next word to guess  
  private lateinit var wordList: MutableList<String>  
  
  companion object {  
  private const val DONE = 0L  
  private const val ONE_SECOND = 1000L //1초  
  private const val COUNTDOWN_TIME = 5000L //5초  
  }  
  
  private val _currentTime = MutableLiveData<Long>()  
  val currentTime : LiveData<Long>  
  get() = _currentTime  
  
    private val timer: CountDownTimer  
  
  val currentTimeString = Transformations.map(currentTime){ time ->  
  DateUtils.formatElapsedTime(time)  
  
  }  
  
  val wordHint = Transformations.map(word){ word ->  
  val randomPosition = (1..word.length).random()  
  "Current word has " + word.length +" letters" +  
  "\n The letter at position" + randomPosition + " is "+  
  word.get(randomPosition)  
  }  
  
  //GameFragment 들어가면 실행  
  init {  
  _word.value = ""  
  _score.value = 0  
  resetList()  
  nextWord()  
  
  timer = object : CountDownTimer(COUNTDOWN_TIME, ONE_SECOND){  
  override fun onTick(p0: Long) {  
  _currentTime.value = p0/ ONE_SECOND  
            }  
  
  override fun onFinish() {  
  _currentTime.value = DONE  
                onEndGame()  
 } }  timer.start()  
  
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
  resetList()  
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
  timer.cancel()  
  
 }  
  fun onEndGame(){  
  _eventGameFinish.value = true  
  }  
  
  fun onEndGameFinished(){  
  _eventGameFinish.value =false  
  }  
  
}
~~~
---
**참고**
https://developer.android.com/codelabs/kotlin-android-training-live-data-transformations#0


 


