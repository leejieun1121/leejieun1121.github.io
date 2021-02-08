---
title : "RecyclerView ViewType에 따라 뷰화면 변경하는 예제 "
excerpt : "viewType 사용해서 리싸이클러뷰 뷰화면 구분하기"

categories:
  - Android
tags :
  - Kotlin
---



예전에 솝트하면서 전체게시판을 구현할때,  viewType 쓰는 방법을 처음 배웠다. 

오랜만에 다시해보고 정리하기  :punch:

![스크린샷 2021-02-08 오후 10 12 42](https://user-images.githubusercontent.com/53978090/107224448-c8005d00-6a5a-11eb-9517-0a0283848f8c.png)

**1.viewType에 따라 보여질 레이아웃 만들기**

위에 보이는 레이아웃은 
1) 가운데 TextView 하나 있는 레이아웃
2) 왼쪽에 몰려있는 레이아웃
3) 오른쪽에 몰려있는 레이아웃 
총 세가지다. 


**item_center.xml**
~~~
<?xml version="1.0" encoding="utf-8"?>  
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"  
  android:layout_width="match_parent"  
  android:layout_height="wrap_content"  
  xmlns:app="http://schemas.android.com/apk/res-auto">  
  
 <TextView  android:id="@+id/content"  
  android:layout_width="wrap_content"  
  android:layout_height="wrap_content"  
  android:layout_marginVertical="20dp"  
  android:text="TextView"  
  android:textSize="20dp"  
  app:layout_constraintBottom_toBottomOf="parent"  
  app:layout_constraintLeft_toLeftOf="parent"  
  app:layout_constraintRight_toRightOf="parent"  
  app:layout_constraintTop_toTopOf="parent" />  
</androidx.constraintlayout.widget.ConstraintLayout>
~~~

**item_left.xml**
~~~
<?xml version="1.0" encoding="utf-8"?>  
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"  
  android:layout_width="match_parent"  
  android:layout_height="wrap_content"  
  xmlns:app="http://schemas.android.com/apk/res-auto">  
 <ImageView  android:id="@+id/imageView"  
  android:layout_width="wrap_content"  
  android:layout_height="wrap_content"  
  android:layout_marginLeft="10dp"  
  android:layout_marginTop="20dp"  
  app:layout_constraintBottom_toBottomOf="parent"  
  app:layout_constraintLeft_toLeftOf="parent"  
  app:layout_constraintTop_toTopOf="parent"  
  app:srcCompat="@mipmap/ic_launcher_round" />  
  
 <TextView  android:id="@+id/name"  
  android:layout_width="wrap_content"  
  android:layout_height="wrap_content"  
  android:layout_marginLeft="10dp"  
  android:layout_marginTop="20dp"  
  android:text="TextView"  
  app:layout_constraintLeft_toRightOf="@id/imageView"  
  app:layout_constraintTop_toTopOf="parent" />  
  
 <TextView  android:id="@+id/content"  
  android:layout_width="wrap_content"  
  android:layout_height="wrap_content"  
  android:layout_marginLeft="10dp"  
  android:layout_marginTop="10dp"  
  android:text="TextView"  
  app:layout_constraintLeft_toRightOf="@id/imageView"  
  app:layout_constraintTop_toBottomOf="@id/name" />  
</androidx.constraintlayout.widget.ConstraintLayout>
~~~

**item_right.xml**

~~~
<?xml version="1.0" encoding="utf-8"?>  
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"  
  android:layout_width="match_parent"  
  android:layout_height="wrap_content"  
  xmlns:app="http://schemas.android.com/apk/res-auto">  
 <ImageView  android:id="@+id/imageView"  
  android:layout_width="wrap_content"  
  android:layout_height="wrap_content"  
  android:layout_marginRight="10dp"  
  android:layout_marginTop="20dp"  
  app:layout_constraintBottom_toBottomOf="parent"  
  app:layout_constraintRight_toRightOf="parent"  
  app:layout_constraintTop_toTopOf="parent"  
  app:srcCompat="@mipmap/ic_launcher_round" />  
  
 <TextView  android:id="@+id/name"  
  android:layout_width="wrap_content"  
  android:layout_height="wrap_content"  
  android:layout_marginRight="10dp"  
  android:layout_marginTop="20dp"  
  android:text="TextView"  
  app:layout_constraintRight_toLeftOf="@id/imageView"  
  app:layout_constraintTop_toTopOf="parent" />  
  
 <TextView  android:id="@+id/content"  
  android:layout_width="wrap_content"  
  android:layout_height="wrap_content"  
  android:layout_marginRight="10dp"  
  android:layout_marginTop="10dp"  
  android:text="TextView"  
  app:layout_constraintRight_toLeftOf="@id/imageView"  
  app:layout_constraintTop_toBottomOf="@id/name" />  
  
</androidx.constraintlayout.widget.ConstraintLayout>
~~~

**2.레이아웃에 넣을 data 만들기**

겉으로 나타나는 사진, 이름 , 내용 그리고 구분하기 위해 필요한 viewType 을 넣어 data를 만들어준다.

**DataItem**
~~~
package com.example.multipleviewtypetest  
  
class DataItem {  
  var content=""  
  var name = ""  
  var picture = ""  
  var viewType = 0  
  
  constructor(content:String, name: String?="", picture : String?="",viewType:Int){  
  this.content = content  
  if (name != null) {  
  this.name = name  
        }  
  if (picture != null) {  
  this.picture = picture  
        }  
  this.viewType = viewType  
  }  
}
~~~

**3.Adapter와 ViewHolder만들기**

리싸이클러뷰를 나타내기 위해서는 Adapter와 ViewHolder가 필요한건 동일하지만 각각 다른 레이아웃을 나타내려면, viewType을 사용해야한다.
~~~
//viewType에 따라 구분해주기 위해 필요, 상수로 선언해줌 ! companion object -> java의 static과 유사  
companion object{  
  const val LEFT_CONTENT = 0  
  const val RIGHT_CONTENT = 1  
  const val CENTER_CONTENT = 2  
}
~~~

동일한 레이아웃의 리싸이클러뷰를 만들때, Adapter의 onCreateViewHolder에서 viewType을 본적이 있을것이다.
이 viewType은 getItemViewType이라는 메소드에서 return 해주는 viewType이다. 

getItemViewType이라는 함수를 오버라이드 한뒤, 아까 만들어둔 DataItem의 viewType을 return 해서 이걸로 구분해줘야한다. :blush: 

~~~
override fun getItemViewType(position: Int): Int {  
  //DataItem타입의 arrayList , 각 position의 viewType에 따라서 레이아웃을 다르게 해줌   
 return myDataList[position].viewType  
}
~~~

onCreateViewHolder에서는 만들어둔 item의 layout을 inflate 해서 viewHolder의 매개변수로 넘겨준다.

아까 세개의 item 레이아웃을 만들었으니, viewType에 따라 3가지 ViewHolder를 만들어준다. 

~~~
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {  
  val inflater = LayoutInflater.from(parent.context)  
  var view : View  
  return when (viewType) {  
  CENTER_CONTENT -> {  
  view = inflater.inflate(R.layout.item_center,parent,false)  
  CenterViewHolder(view)  
 }  LEFT_CONTENT -> {  
  view = inflater.inflate(R.layout.item_left,parent,false)  
  LeftViewHolder(view)  
 }  else -> {  
  view = inflater.inflate(R.layout.item_right,parent,false)  
  RightViewHolder(view)  
 } }}
~~~

그다음 onBindViewHolder에서 holder의 종류에 따라 bind를 해준다.

여기서 set을 해줘도 되지만, 나는 viewHolder에 bind함수를 만들어 set해줬다. 

~~~
override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {  
  when (holder) {  
  is CenterViewHolder -> {  
  holder.bind(myDataList[position])  
 }  is LeftViewHolder -> {  
  holder.bind(myDataList[position])  
 }  else -> { // 무슨 viewHolder인지 제대로 안정해줬으니까, as로 정해주기  
  (holder as RightViewHolder).bind(myDataList[position])  
  
 } }}
~~~

그다음 각 ViewHolder에서 findViewById를 사용해서 set할 아이템들을 찾아주고, adapter 에서 호출했던 bind 함수안에서 데이터를 set해주면 된다.

~~~
class CenterViewHolder(itemView : View) : RecyclerView.ViewHolder(itemView) {  
  val content: TextView = itemView.findViewById(R.id.content)  
  
  fun bind(dataItem:DataItem){  
  content.text = dataItem.content  
    }  
}
~~~ 

~~~
class LeftViewHolder(itemView : View) : RecyclerView.ViewHolder(itemView) {  
  val content: TextView = itemView.findViewById(R.id.content)  
  val name: TextView = itemView.findViewById(R.id.name)  
  val image: ImageView = itemView.findViewById(R.id.imageView)  
  
  fun bind(dataItem:DataItem){  
  content.text = dataItem.content  
        name.text= dataItem.name  
        Glide.with(image.context).load(dataItem.picture).override(300,300).into(image)  
 }}
~~~

~~~
class RightViewHolder(itemView : View) : RecyclerView.ViewHolder(itemView) {  
  val content: TextView = itemView.findViewById(R.id.content)  
  val name: TextView = itemView.findViewById(R.id.name)  
  val image: ImageView = itemView.findViewById(R.id.imageView)  
  
  fun bind(dataItem:DataItem){  
  content.text = dataItem.content  
        name.text= dataItem.name  
        Glide.with(image.context).load(dataItem.picture).override(300,300).into(image)  
 }}
~~~

**MyAdapter.kt**
~~~
package com.example.multipleviewtypetest  
  
import android.util.Log  
import android.view.LayoutInflater  
import android.view.View  
import android.view.ViewGroup  
import android.widget.ImageView  
import android.widget.TextView  
import androidx.recyclerview.widget.RecyclerView  
import com.bumptech.glide.Glide  
  
class MyAdapter : RecyclerView.Adapter<RecyclerView.ViewHolder> {  
  //viewType에 따라 구분해주기 위해 필요, 상수로 선언해줌 ! companion object -> java의 static과 유사  
  companion object{  
  const val LEFT_CONTENT = 0  
  const val RIGHT_CONTENT = 1  
  const val CENTER_CONTENT = 2  
 }  private  var myDataList = arrayListOf<DataItem>()  
  
  constructor(dataList:ArrayList<DataItem>?)  
 {  if (dataList != null) {  
  this.myDataList = dataList  
        }  
 }  
  override fun getItemViewType(position: Int): Int {  
  //DataItem타입의 arrayList , 각 position의 viewType에 따라서 레이아웃을 다르게 해줌  
  return myDataList[position].viewType  
    }  
  
  override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {  
  val inflater = LayoutInflater.from(parent.context)  
  var view : View  
  return when (viewType) {  
  CENTER_CONTENT -> {  
  view = inflater.inflate(R.layout.item_center,parent,false)  
  CenterViewHolder(view)  
 }  LEFT_CONTENT -> {  
  view = inflater.inflate(R.layout.item_left,parent,false)  
  LeftViewHolder(view)  
 }  else -> {  
  view = inflater.inflate(R.layout.item_right,parent,false)  
  RightViewHolder(view)  
 } } }  
  override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {  
  when (holder) {  
  is CenterViewHolder -> {  
  holder.bind(myDataList[position])  
 }  is LeftViewHolder -> {  
  holder.bind(myDataList[position])  
 }  else -> { // 무슨 viewHolder인지 제대로 안정해줬으니까, as로 정해주기  
  (holder as RightViewHolder).bind(myDataList[position])  
  
 } } }  
  override fun getItemCount(): Int {  
  return myDataList.size  
    }  
  
}  
  
class CenterViewHolder(itemView : View) : RecyclerView.ViewHolder(itemView) {  
  val content: TextView = itemView.findViewById(R.id.content)  
  
  fun bind(dataItem:DataItem){  
  content.text = dataItem.content  
    }  
}  
  
class LeftViewHolder(itemView : View) : RecyclerView.ViewHolder(itemView) {  
  val content: TextView = itemView.findViewById(R.id.content)  
  val name: TextView = itemView.findViewById(R.id.name)  
  val image: ImageView = itemView.findViewById(R.id.imageView)  
  
  fun bind(dataItem:DataItem){  
  content.text = dataItem.content  
        name.text= dataItem.name  
        Glide.with(image.context).load(dataItem.picture).override(300,300).into(image)  
 }}  
  
class RightViewHolder(itemView : View) : RecyclerView.ViewHolder(itemView) {  
  val content: TextView = itemView.findViewById(R.id.content)  
  val name: TextView = itemView.findViewById(R.id.name)  
  val image: ImageView = itemView.findViewById(R.id.imageView)  
  
  fun bind(dataItem:DataItem){  
  content.text = dataItem.content  
        name.text= dataItem.name  
        Glide.with(image.context).load(dataItem.picture).override(300,300).into(image)  
 }}
~~~

**4. 리싸이클러뷰 adapter설정 및 데이터 넣기**

리싸이클러뷰가 포함된 레이아웃을 만들어준다.

**activity_main.xml**

~~~
<?xml version="1.0" encoding="utf-8"?>  
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"  
  xmlns:app="http://schemas.android.com/apk/res-auto"  
  xmlns:tools="http://schemas.android.com/tools"  
  android:layout_width="match_parent"  
  android:layout_height="match_parent"  
  tools:context=".MainActivity">  
  
 <androidx.recyclerview.widget.RecyclerView  android:id="@+id/rv_chatting"  
  android:layout_width="match_parent"  
  android:layout_height="match_parent"/>  
  
</androidx.constraintlayout.widget.ConstraintLayout>
~~~

만들어둔 리싸이클러뷰에 레이아웃 메니저와 어댑터를 설정해주고, 데이터들을 추가시키면 끝 !

**MainActivity.kt**

~~~
package com.example.multipleviewtypetest  
  
import androidx.appcompat.app.AppCompatActivity  
import android.os.Bundle  
import androidx.recyclerview.widget.LinearLayoutManager  
import androidx.recyclerview.widget.RecyclerView  
import com.example.multipleviewtypetest.MyAdapter.Companion.CENTER_CONTENT  
import com.example.multipleviewtypetest.MyAdapter.Companion.LEFT_CONTENT  
import com.example.multipleviewtypetest.MyAdapter.Companion.RIGHT_CONTENT  
  
class MainActivity : AppCompatActivity() {  
  private var dataList = arrayListOf<DataItem>()  
  override fun onCreate(savedInstanceState: Bundle?) {  
  super.onCreate(savedInstanceState)  
  setContentView(R.layout.activity_main)  
  
  addItem()  
  //리싸이클러뷰에 레이아웃 매니저, 어댑터 설정해주기   
 val rvChatting = findViewById<RecyclerView>(R.id.rv_chatting)  
  val layoutManager = LinearLayoutManager(this,LinearLayoutManager.VERTICAL,false)  
  rvChatting.layoutManager = layoutManager  
        //아이템 추가한 리스트 Adapter에 넘겨주기    
 rvChatting.adapter = MyAdapter(dataList)  
  
 }  
  //리싸이클러뷰 아이템 추가   
 private fun addItem(){  
  dataList.add(DataItem("사용자1님이 입장하셨습니다.",null,"",CENTER_CONTENT))  
  dataList.add(DataItem("사용자2님이 입장하셨습니다.",null,"",CENTER_CONTENT))  
  dataList.add(DataItem("안녕하세요","사용자1","https://image.freepik.com/free-photo/friendly-smart-basenji-dog-giving-his-paw-close-up-isolated-white_346278-1626.jpg",LEFT_CONTENT))  
  dataList.add(DataItem("안녕하세요","사용자2","https://image.freepik.com/free-photo/french-bulldog-dog-breeds-white-polka-dot-black-marble_1150-25345.jpg",RIGHT_CONTENT))  
  
 }}
~~~



---
**참고**

<https://lktprogrammer.tistory.com/190>

<https://www.freepik.com/free-photo/french-bulldog-dog-breeds-white-polka-dot-black-marble_10069489.htm#position=17>
