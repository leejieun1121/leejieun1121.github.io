---
title : "Android CustomView 만들기(kotlin) "
excerpt : "Android CustomView"

categories:
  - Android
tags :
  - Android 
  - Kotlin
---

## Android CustomView 만들기 (Kotlin)



앱 레이아웃을 만들 때, 똑같은 레이아웃이 많아 복사해서 붙여넣는 경우가 많다.

하지만 코드를 복붙하면 지저분해지고 보기도 힘들어진다. 이럴때 겹치는 레이아웃을 커스텀 뷰로 만들어서 사용하면 훨씬 간편해진다 !

![스크린샷 2021-01-22 오후 5 12 49](https://user-images.githubusercontent.com/53978090/105466472-9c017f80-5cd7-11eb-829a-b72b548a01c1.png)



LinearLayout 안에 Imageview 와 TextView 가 있는 모습이다. 

color 값 찾기 귀찮아서 원래 있는거 썼더니 조합이 너무 별로다 ㅎ.. 

---



1. 재사용 할 레이아웃을 만든다. 

   < customview.xml >

   '''
   <?xml version="1.0" encoding="utf-8"?>
   <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
       android:id="@+id/layout"
       android:layout_width="match_parent"
       android:layout_height="60dp"
       android:padding="10dp"
       android:orientation="horizontal"
       >
       <ImageView
           android:id="@+id/imgIcon"
           android:layout_width="50dp"
           android:layout_height="50dp"
           android:layout_marginLeft="20dp"
           android:layout_gravity="center"
           />
   
       <TextView
           android:id="@+id/tvName"
           android:layout_width="0dp"
           android:layout_height="match_parent"
           android:layout_marginLeft="10dp"
           android:textSize="18sp"
           android:paddingTop="5dp"
           android:paddingLeft="5dp"
           android:layout_gravity="center"
           android:layout_marginRight="20dp"
           android:layout_weight="1"
           android:text="asdfaf"
           />
   
   
   </LinearLayout>
   '''



2. values 밑에 CustomView에서 사용할 속성을 추가해준다.

   < attrs.xml >

   '''
   <?xml version="1.0" encoding="utf-8"?>
   <resources>
       <declare-styleable name="LoginButton">
           <attr name="bgColor" format="reference|integer" />
           <attr name="imgColor" format="reference|integer" />
           <attr name="text" format="reference|string" />
           <attr name="textColor" format="reference|integer" />
       </declare-styleable>
   </resources>재사용 할 레이아웃을 만든다. 
   '''

   여기서 쓰이는 name = "LoginButton"이 3.에서 쓰이는 styleable 의 이름이 된다.

   

3. 재사용 할 레이아웃을 커스텀 뷰로 만드는 코드 작성

   < CustomLoginButtom.kt >

   '''
   package com.example.customview
   
   import android.content.Context
   import android.content.res.TypedArray
   import android.util.AttributeSet
   import android.util.Log
   import android.view.LayoutInflater
   import android.widget.ImageView
   import android.widget.LinearLayout
   import android.widget.TextView
   
   /*
   //밑의 생성자 3개를 간단하게 이렇게 쓸 수 있다.
   class CustomView @JvmOverloads
   constructor(context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0)
   : View(context, attrs, defStyleAttr) {
   
       // ...
   }
   */
   
   class CustomLoginButton: LinearLayout {
       //커스텀 뷰 안에 들어가는 아이템들
       lateinit var layout : LinearLayout
       lateinit var tvName : TextView
       lateinit var imgIcon : ImageView
   
       constructor(context: Context?) : super(context){
           init(context)
       }
       constructor(context: Context?, attrs: AttributeSet?) : super(context, attrs){
           init(context)
           getAttrs(attrs)
   
       }
       constructor(context: Context?, attrs: AttributeSet?, defStyleAttr: Int) : super(context, attrs, defStyleAttr){
           init(context)
           getAttrs(attrs,defStyleAttr)
       }
   
       //아까 만들어둔 커스텀뷰 레이아웃을 inflate 해서 적용해준다.
       //그리고 layout에서 id를 참조해 위에 lateinit 으로 냅뒀던 변수들을 초기화 해준다.
       private fun init(context:Context?){
           val view = LayoutInflater.from(context).inflate(R.layout.customview,this,false)
           addView(view)
   
           layout = findViewById(R.id.layout)
           tvName = findViewById(R.id.tvName)
           imgIcon = findViewById(R.id.imgIcon)
       }
       
       private fun getAttrs(attrs:AttributeSet?){
           //아까 만들어뒀던 속성 attrs 를 참조함 
           val typedArray = context.obtainStyledAttributes(attrs,R.styleable.LoginButton)
           setTypeArray(typedArray)
       }
   
       private fun getAttrs(attrs:AttributeSet?, defStyle:Int){
           val typedArray = context.obtainStyledAttributes(attrs,R.styleable.LoginButton,defStyle,0)
           setTypeArray(typedArray)
       }
       
       //디폴트 설정 
       private fun setTypeArray(typedArray : TypedArray){
           //레이아웃의 배경, LoginButton 이름으로 만든 attrs.xml 속성중 bgColor 를 참조함  
           val bgResId = typedArray.getResourceId(R.styleable.LoginButton_bgColor,R.drawable.ic_launcher_background)
           layout.setBackgroundResource(bgResId)
   
           //이미지의 배경, LoginButton 이름으로 만든 attrs.xml 속성중 imgColor 를 참조함
           val imgResId = typedArray.getResourceId(R.styleable.LoginButton_imgColor,R.drawable.ic_launcher_foreground)
           imgIcon.setBackgroundResource(imgResId)
   
           //텍스트 색, LoginButton 이름으로 만든 attrs.xml 속성중 textColor 를 참조함
           val textColor = typedArray.getColor(R.styleable.LoginButton_textColor,0)
           tvName.setTextColor(textColor)
   
           //텍스트 내용, LoginButton 이름으로 만든 attrs.xml 속성중 text 를 참조함
           val text = typedArray.getText(R.styleable.LoginButton_text)
           tvName.text = text
   
           typedArray.recycle()
       }
   }
   
   '''



4. 커스텀뷰를 쓰려는 레이아웃에서 만들어둔 CustomLoginButton을 사용하면 된다.

   < activity_main.xml >

   '''
   <?xml version="1.0" encoding="utf-8"?>
   <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
       xmlns:app="http://schemas.android.com/apk/res-auto"
       xmlns:tools="http://schemas.android.com/tools"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       android:orientation="vertical"
       tools:context=".MainActivity">
   
       <com.example.customview.CustomLoginButton
           android:layout_width="match_parent"
           android:layout_height="wrap_content"
           app:bgColor = "@color/black"
           app:imgColor = "@drawable/ic_baseline_assistant_24"
           app:text="텍스트"
           app:textColor="@color/teal_200"
           />
   
       <com.example.customview.CustomLoginButton
           android:layout_width="match_parent"
           android:layout_height="wrap_content"
           app:bgColor = "@color/white"
           app:imgColor = "@drawable/ic_baseline_attachment_24"
           app:text="텍스트2222"
           app:textColor="@color/purple_500"
           />
   
       <com.example.customview.CustomLoginButton
           android:layout_width="match_parent"
           android:layout_height="wrap_content"
           app:bgColor="@color/purple_200"
           app:text="텍스트33"
           app:textColor="@color/white"
           />
   
   </LinearLayout>
   '''



이렇게 아까 attrs.xml 에 썼던 속성들을 이용해서 Custom 시킬 수 있다. 

아쉬운점은 CustomView의 속성들을 바로바로 찾아주지 않아 일일이 쳐야하는 점이 귀찮다 ㅠ 

---
** 참고 **

https://gun0912.tistory.com/38

https://dunkey2615.tistory.com/54
