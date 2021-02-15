---
title : "Navigation 연습1 (kotlin) "
excerpt : "코드랩 따라하면서 공부하기~"

categories:
  - Android
tags :
  - Kotlin
---


### Navigation 연습 1 

**라이브러리추가**
< 프로젝트 단위 build.gradle >
~~~
ext {               navigationVersion = "2.3.0"          }
~~~

< 모듈 단위 build.gradle > 
~~~
dependencies {   implementation "androidx.navigation:navigation-fragment-ktx:$navigationVersion"  implementation "androidx.navigation:navigation-ui-ktx:$navigationVersion" }
~~~

**Navigation 추가**
res 우클릭 -> New -> Anrdroid Resource File , Resource type Navigation 선택하기 

**MainActivity에 NavigationFragment 붙이기**

< activity_main.xml >
```
<?xml version="1.0" encoding="utf-8"?><!--  
 ~ Copyright 2018, The Android Open Source Project ~ ~ Licensed under the Apache License, Version 2.0 (the "License"); ~ you may not use this file except in compliance with the License. ~ You may obtain a copy of the License at ~ ~     http://www.apache.org/licenses/LICENSE-2.0 ~ ~ Unless required by applicable law or agreed to in writing, software ~ distributed under the License is distributed on an "AS IS" BASIS, ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. ~ See the License for the specific language governing permissions and ~ limitations under the License. -->  
<layout xmlns:android="http://schemas.android.com/apk/res/android"  
  xmlns:app="http://schemas.android.com/apk/res-auto">  
  
 <LinearLayout  android:layout_width="match_parent"  
  android:layout_height="match_parent"  
  android:orientation="vertical">  
  
 <fragment  android:id="@+id/myNavHostFragment"  
  android:name="androidx.navigation.fragment.NavHostFragment"  
  android:layout_width="match_parent"  
  android:layout_height="match_parent"  
  app:navGraph="@navigation/navigation"  
  app:defaultNavHost="true" />  
 </LinearLayout>  
</layout>
```

**아까 만든 navigation.xml 에 이동할 fragment 추가** 
<img width="279" alt="스크린샷 2021-01-29 오후 3 52 06" src="https://user-images.githubusercontent.com/53978090/106242096-81388900-624a-11eb-8c71-6121fdafbca3.png">

집모양 아이콘이 출발 fragment이고 다음으로 이동할 Fragment를 추가해서 방향 설정을 해준다.

**버튼 누르면 다음 fragment 로 이동하기**

< TitleFragment.kt >
```
class TitleFragment : Fragment() {  
  lateinit var binding : FragmentTitleBinding  
  
  override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,  
                              savedInstanceState: Bundle?): View? {  
  // Inflate the layout for this fragment  
  binding = DataBindingUtil.inflate(inflater,R.layout.fragment_title,container,false)  
  
  binding.playButton.setOnClickListener {  
  findNavController().navigate(R.id.action_titleFragment_to_gameFragment)  
  }  
  
  return binding.root  
  
  
  }  
  
  
}
```
navitation.xml에서 TitleFragment -> GameFragment 로 연결시켜놓으면
action이 생기는데, 그 action의 id를 적어 
findNavController(). navigate(R.id~~) 해주면 이동하게 된다.

![ezgif com-gif-maker (5)](https://user-images.githubusercontent.com/53978090/106242875-babdc400-624b-11eb-96bb-7cc08604cfed.gif)

---
**back 버튼 처리하기**
원래 back 버튼을 누르면 백스택의 맨 위(마지막으로 본 화면) 으로 넘어간다. 하지만 원하는 화면으로 back 하기 위해서는 pop Behavior 를 정해줘야한다. 

navigation 의 Design 으로 들어가서 

<img width="696" alt="스크린샷 2021-01-29 오후 4 25 36" src="https://user-images.githubusercontent.com/53978090/106244680-a3cca100-624e-11eb-93ef-4f5989fe825b.png">

Pop Behavior -> Pop To를 어떤 fragment 로 할지 정해준다. 
Inclusive를 true 로 해주면 백스택에서 삭제될 수 있음을 의미한다! 

ex) 만약 fragment 진행 순서가 title -> game -> won 이라면 back 을 눌렀을때, 
won -> game -> title 순으로 이동할것이다.(스택에 쌓인 순서대로)

하지만 popUpTo 대상이 title 이고  Inclusive = true라면, title 으로 이동하면서 이전의 쌓아놨던 won , game이  pop 되어 스택이 비워지게 된다. 

그리고 fragment 에 다음과 같은 코드를 추가해주면 된다.
~~~
override fun onCreate(savedInstanceState: Bundle?) {  
  super.onCreate(savedInstanceState)  
  val callback = requireActivity().onBackPressedDispatcher.addCallback(this){  
  findNavController().navigate(R.id.action_gameOverFragment_to_titleFragment)  
  }  
}
~~~

![ezgif com-gif-maker (6)](https://user-images.githubusercontent.com/53978090/106251277-227a0c00-6258-11eb-985a-7139c48c1457.gif)

---
**Up button 만들기**

< MainActivity.kt >
~~~
class MainActivity : AppCompatActivity() {  
  override fun onCreate(savedInstanceState: Bundle?) {  
  super.onCreate(savedInstanceState)  
  @Suppress("UNUSED_VARIABLE")  
  val binding = DataBindingUtil.setContentView<ActivityMainBinding>(this, R.layout.activity_main)  
  
  val navController = this.findNavController(R.id.myNavHostFragment)  
  NavigationUI.setupActionBarWithNavController(this,navController)  
 }  
  override fun onSupportNavigateUp(): Boolean {  
  val navController = this.findNavController(R.id.myNavHostFragment)  
  return navController.navigateUp()  
 }}
~~~

---
**Navigation 옵션 메뉴 추가**

먼저, Navigation에 메뉴 아이템을 클릭했을때 이동할 fragment 를 추가해준다.
(navigation.xml에서 해당 fragment의 id를 확인) 

res->menu에서 메뉴를 생성해주고
Menu Item을 추가한다.
id와 title을 추가해준다. 
- id : 이동할 fragment 와 일치해야함! 
- title : 이동했을때 상단바에 뜨는 이름 

옵션 메뉴를 보이게 할 fragment에
다음과 같은 코드를 작성한다. 

 < options_menu.xml >
 ~~~
 <?xml version="1.0" encoding="utf-8"?>
<menu xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:android="http://schemas.android.com/apk/res/android">

    <item
        android:id="@+id/fragment_about"
        android:title="@string/about" />
</menu>
 ~~~

 < TitleFragment.kt >
  ~~~
 class TitleFragment : Fragment() {  
  lateinit var binding : FragmentTitleBinding  
  
  override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,  
                              savedInstanceState: Bundle?): View? {  
  // Inflate the layout for this fragment  
  binding = DataBindingUtil.inflate(inflater,R.layout.fragment_title,container,false)  
  
  binding.playButton.setOnClickListener {  
  findNavController().navigate(R.id.action_titleFragment_to_gameFragment)  
  }  
  
  //옵션메뉴 보이게 함   
 setHasOptionsMenu(true)  
  return binding.root  
  
  
  }  
  
  //만들어둔 xml로 옵션 메뉴를 만듦   
 override fun onCreateOptionsMenu(menu: Menu, inflater: MenuInflater) {  
  super.onCreateOptionsMenu(menu, inflater)  
  inflater.inflate(R.menu.options_menu, menu)  
 }  
  //메뉴의 아이템이 눌렸을때 해당 fragment 로 이동   
 override fun onOptionsItemSelected(item: MenuItem): Boolean {  
  return NavigationUI.  
        onNavDestinationSelected(item,requireView().findNavController())  
  || super.onOptionsItemSelected(item)  
 }  
}
 ~~~
 
![ezgif com-gif-maker (7)](https://user-images.githubusercontent.com/53978090/106254957-ad5d0580-625c-11eb-8ce3-c8cdcf831eb8.gif)
 
 ---
**Drawer Layout 추가하기**
옵션 메뉴만들때 처럼 menu를 추가하고
activity_main.xml에 DrawerLayout으로 전체를 감싸고 NavigationView의 menu에 만들어둔 menu를 넣는다.  
< activity_main.xml >
~~~
<layout xmlns:android="http://schemas.android.com/apk/res/android"  
  xmlns:app="http://schemas.android.com/apk/res-auto">  
  
 <androidx.drawerlayout.widget.DrawerLayout  android:id="@+id/layout_drawer"  
  android:layout_width="match_parent"  
  android:layout_height="match_parent">  
  
 <LinearLayout  android:layout_width="match_parent"  
  android:layout_height="match_parent"  
  android:orientation="vertical">  
  
 <fragment  android:id="@+id/myNavHostFragment"  
  android:name="androidx.navigation.fragment.NavHostFragment"  
  android:layout_width="match_parent"  
  android:layout_height="match_parent"  
  app:navGraph="@navigation/navigation"  
  app:defaultNavHost="true" />  
 </LinearLayout>  
 <com.google.android.material.navigation.NavigationView  android:id="@+id/navView"  
  android:layout_width="wrap_content"  
  android:layout_height="match_parent"  
  android:layout_gravity="start"  
  app:headerLayout="@layout/nav_header"  
  app:menu="@menu/navdrawer_menu" />  
  
 </androidx.drawerlayout.widget.DrawerLayout>  
</layout>
~~~

< MainActivity.kt >
~~~
private lateinit var drawerLayout: DrawerLayout

NavigationUI.setupWithNavController(binding.navView, navController)  
drawerLayout = binding.layoutDrawer  
NavigationUI.setupActionBarWithNavController(this, navController, drawerLayout)
~~~
~~~
override fun onSupportNavigateUp(): Boolean {  
  val navController = this.findNavController(R.id.myNavHostFragment)  
  return NavigationUI.navigateUp(navController, drawerLayout)  
}
~~~
![ezgif com-gif-maker (8)](https://user-images.githubusercontent.com/53978090/106260133-6a526080-6263-11eb-8ea1-44e457774087.gif)

---

**참고**
코드랩 :<https://developer.android.com/codelabs/kotlin-android-training-add-navigation/index.html#0>

Android developvers: <https://developer.android.com/guide/navigation/navigation-navigate?hl=ko>
