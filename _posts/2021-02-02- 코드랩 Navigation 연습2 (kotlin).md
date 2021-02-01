---
title : "Navigation 연습2 (kotlin) "
excerpt : "코드랩 따라하면서 공부하기~"

categories:
  - Android
tags :
  - Kotlin
---

### Navigation 연습 2

>Fragment 간의 값을 전달하는 방법

#### 1. Bundle 
: key-value 형식으로 이루어져있음

#### 2. Safe Args

이렇게 두가지가 있다. 하지만
Bundle을 사용하면 다음과 같은 오류가 발생할 수 있다.
1) 유형 불일치
ex) 정수형 보냈는데 string 으로 받기 
2) 키 오류 null값 
ex) 없는 키를 사용했을때  

그래서 fragment간의 데이터를 전달할 때, SafeArgs를 사용하는걸 권장한다고 한다. 

---
 
 #### SafeArgs 시작하기

< build.gradle > 프로젝트 수준
~~~
classpath "androidx.navigation:navigation-safe-args-gradle-plugin:$navigationVersion"
~~~

< build.gradle > 모듈 수준

```
apply plugin: 'androidx.navigation.safeargs'
```
 
 두 개의 라이브러리를 추가하고 나면
NavDirection클래스가 포함된다.
ex) GameFragment -> GameFragmentDirection 

그리고 navigation.xml에 들어가서 넘겨받을 인수를 설정해준다.

<img width="868" alt="스크린샷 2021-02-01 오후 7 43 37" src="https://user-images.githubusercontent.com/53978090/106448417-15625480-64c6-11eb-859e-2250d8bb48fb.png">

Name과 Type으로 넘겨받을 데이터의 이름, 타입을 설정해주면 된다.

그리고 값을 넘길 fragment 에서 
~~~
findNavController().navigate(GameFragmentDirections.actionGameFragmentToGameWonFragment(numQuestions,questionIndex))
~~~
만들어진 Directions을 통해서 두개의 데이터를 넘겨준다. (위에서 설정한 넘겨받을 데이터의 개수와 타입이 맞아야함 !)

마지막으로 받는쪽 fragment에서 
~~~
val args = GameWonFragmentArgs.fromBundle(requireArguments())
Toast.makeText(context, "NumCorrect: ${args.numCorrect}, NumQuestions: ${args.numQuestions}", Toast.LENGTH_LONG).show()
~~~
args.아까 설정한 데이터의 이름 이렇게 접근해서 넘겨받은 값을 확인할 수 있다. 

![스크린샷 2021-02-02 오전 1 06 13](https://user-images.githubusercontent.com/53978090/106484343-e0202b80-64f2-11eb-9e6d-a7235760511b.png)

> 암시적인텐트

- 명시적 인텐트 
 : 특정 컴포넌트나 액티비티가 명확하게 실행되어야 할 경우
 ex) 평소에 Activity 끼리 Intent이동할때 사용했던 코드들 , 옆에 어떤 Activity로 넘길지 써줌 
 intent.putExtra~~ 이런식
- 암시적 인텐트
: 호출할 대상이 달라질 수 있는 경우, 의도에 맞는 적당한 액티비티를 찾아 실행시켜줌 
ex) 선택지가 생기는 경우
 공유 앱 설정할때 여러개 선택할 수 있는것 처럼 어떤것을 사용할지 물어봄 !  

#### 공유 기능 만들기

onCreateView 메소드에  setHasOptionsMenu(true)를 적어주고

미리 만들어둔 share menu 를 이용해서 메뉴를 만들어 클릭시 공유할 수 있는 여러 위젯들이 뜬다. 

~~~
override fun onCreateOptionsMenu(menu: Menu, inflater: MenuInflater) {  
  super.onCreateOptionsMenu(menu, inflater)  
  inflater.inflate(R.menu.winner_menu, menu)  
  //resolveActivity = 실행하기 위해 가장 적합한 컴포넌트가 없다면, 아예 메뉴에서 보이지 않도록   
 if(getShareIntent().resolveActivity(requireActivity().packageManager)==null){  
  menu.findItem(R.id.share).isVisible = false  
  }  
}  
  
// Sharing from the Menu  
override fun onOptionsItemSelected(item: MenuItem): Boolean {  
  when(item.itemId){  
  R.id.share -> shareSuccess()  
 }  return super.onOptionsItemSelected(item)  
}  
  
// Creating our Share Intent  
private fun getShareIntent() : Intent {  
  //넘겨받은 인수를 shareIntent에 set val args = GameWonFragmentArgs.fromBundle(requireArguments())  
  val shareIntent = Intent(Intent.ACTION_SEND)  
  shareIntent.setType("text/plain")  
  .putExtra(Intent.EXTRA_TEXT, getString(R.string.share_success_text, args.numCorrect, args.numQuestions))  
  return shareIntent  
}  
  
private fun shareSuccess() {  
  //선택한 컴포넌트를 통해 액티비티로 넘어가 넘겨받은 인수를 공유   
 startActivity(getShareIntent())  
}
~~~


![스크린샷 2021-02-02 오전 1 04 30](https://user-images.githubusercontent.com/53978090/106484107-9df6ea00-64f2-11eb-881d-b68b58d162a7.png)
	
---
**정리**
단순히 navigation 화면 넘어간다
~~~
findNavController().navigate(R.id.action_gameWonFragment_to_gameFragment)
~~~
-> 생성된 화살표의 id만 써줘도 됨 

인수를 넘겨야한다
~~~
findNavController().navigate(GameFragmentDirections.actionGameFragmentToGameWonFragment(numQuestions,questionIndex))
~~~
-> Direction을 사용해서 인수를 넣어줘야함 , 인수를 안넣어주면 위와 동일하게 넘어가는듯 !

---
**참고**

코드랩 : <https://developer.android.com/codelabs/kotlin-android-training-start-external-activity/index.html#0>

인텐트 :<https://developer.android.com/training/basics/intents/sending>

<https://limkydev.tistory.com/35>





