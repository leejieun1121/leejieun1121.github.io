---
title : "RecyclerView 아이템 클릭시 해당 아이템 정보 보여주는 예제 "
excerpt : "interface와 listener를 사용한 클릭 이벤트 처리"

categories:
  - Android
tags :
  - Kotlin
---


요즘 리싸이클러뷰를 계속 건드리고있다. 수도없이 만들었지만 가물가물한것도 많고, 여러번 연습해서 내걸로 만들기 ㅎㅎ

리싸이클러뷰는 스크롤하면서 리스트 내용을 보여주는 역할이 가장 크다. 리싸이클러뷰의 각 아이템을 클릭했을때 그 정보를 얻으려면 interface와 listener를 통해 position 정보를 가져오면 된다.

다음은 리싸이클러뷰 클릭시, 해당 position의 아이템 정보를 다른 액티비티에서 보여주는 예제이다.

![ezgif com-gif-maker (10)](https://user-images.githubusercontent.com/53978090/107545181-59640080-6c0e-11eb-9c44-3ff153964dc3.gif)

[앨범사진출처](https://www.discogs.com/ko/)는 여기 !

앨범사진과 노래제목 그리고 가수를 GridLayout으로 보여주는 리싸이클러뷰이다.

**1. RecyclerView 추가하기**

리싸이클러뷰를 나타낼 곳에 RecyclerView를 추가해준다.

**activity_main.xml**
~~~
<?xml version="1.0" encoding="utf-8"?>  
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"  
  xmlns:app="http://schemas.android.com/apk/res-auto"  
  xmlns:tools="http://schemas.android.com/tools"  
  android:layout_width="match_parent"  
  android:layout_height="match_parent"  
  tools:context=".MainActivity">  
  
 <androidx.recyclerview.widget.RecyclerView  android:id="@+id/rv_music_list"  
  android:layout_height="match_parent"  
  android:layout_width="match_parent">  
  
 </androidx.recyclerview.widget.RecyclerView>  
</androidx.constraintlayout.widget.ConstraintLayout>
~~~

**2. 리싸이클러뷰에 나타낼 아이템 레이아웃을 만든다.**

**item_music.xml**
~~~
<?xml version="1.0" encoding="utf-8"?>  
<androidx.constraintlayout.widget.ConstraintLayout  
  xmlns:android="http://schemas.android.com/apk/res/android"  
  xmlns:app="http://schemas.android.com/apk/res-auto"  
  android:layout_width="match_parent"  
  android:layout_height="wrap_content">  
  
 <ImageView  android:id="@+id/img_photo"  
  android:layout_width="150dp"  
  android:layout_height="150dp"  
  app:layout_constraintEnd_toEndOf="parent"  
  app:layout_constraintStart_toStartOf="parent"  
  app:layout_constraintTop_toTopOf="parent" />  
  
 <TextView  android:id="@+id/tv_title"  
  android:layout_marginTop="5dp"  
  android:layout_width="wrap_content"  
  android:layout_height="wrap_content"  
  app:layout_constraintEnd_toEndOf="@+id/img_photo"  
  app:layout_constraintStart_toStartOf="@+id/img_photo"  
  app:layout_constraintTop_toBottomOf="@+id/img_photo" />  
  
 <TextView  android:id="@+id/tv_singer"  
  android:layout_marginTop="5dp"  
  android:layout_width="wrap_content"  
  android:layout_height="wrap_content"  
  app:layout_constraintEnd_toEndOf="@+id/tv_title"  
  app:layout_constraintStart_toStartOf="@+id/tv_title"  
  app:layout_constraintTop_toBottomOf="@+id/tv_title"  
  />  
  
  
</androidx.constraintlayout.widget.ConstraintLayout>
~~~

**item에 설정할 데이터 클래스를 만들어준다.**

다른 액티비티로 객체를 넘길것이기 때문에 Serializable을 상속해줘야한다.

**ItemMusic**
~~~
package com.example.recyclerviewclickitem  
  
import java.io.Serializable  
  
  
data class ItemMusic(val title:String="",val singer:String="",val photo:String="") : Serializable
~~~

**Adapter와 ViewHolder를 만든다.**
 
~~~
class MusicAdapter(list: ArrayList<ItemMusic>) : RecyclerView.Adapter<ViewHolder>() {  
  val musicList = list  
  override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {  
  val inflater = LayoutInflater.from(parent.context)  
  val view = inflater.inflate(R.layout.item_music,parent,false)  
  return ViewHolder(view)  
 }  
  override fun onBindViewHolder(holder: ViewHolder, position: Int) {  
  return holder.bind(musicList[position])  
    }  
  
  override fun getItemCount(): Int {  
  return musicList.size  
    }  
}  
  
class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView){  
  val tvTitle: TextView = itemView.findViewById(R.id.tv_title)  
  val tvSinger:TextView = itemView.findViewById(R.id.tv_singer)  
  val imgPhoto :ImageView= itemView.findViewById(R.id.img_photo)  
  
  fun bind(item:ItemMusic,listener: onClickMusic){  
  tvTitle.text = item.title  
        tvSinger.text = item.singer  
        Glide.with(itemView.context).load(item.photo).into(imgPhoto)  
     }  
}
~~~

**Activity , Fragment설정**

리싸이클러뷰를 찾아 만들어둔 어댑터, 레이아웃 매니저를 설정해준다. 그리고 데이터 추가하기

**MainActivity.kt**
~~~
class MainActivity : AppCompatActivity()  {  
  private val musicList = ArrayList<ItemMusic>()  
  override fun onCreate(savedInstanceState: Bundle?) {  
  super.onCreate(savedInstanceState)  
  setContentView(R.layout.activity_main)  
  
  addItem()  
  val adapter = MusicAdapter(musicList,this)  
  val rvMusicList = findViewById<RecyclerView>(R.id.rv_music_list)  
  val layoutManager = GridLayoutManager(this,2)  
  
  rvMusicList.adapter = adapter  
        rvMusicList.layoutManager = layoutManager  
  
    }  
  
  private fun addItem(){  
  musicList.add(ItemMusic("Dancing In Tne Monnlight","King Harvest","https://img.discogs.com/0s5KntsJ1ei0yFcZXFLzEsBUatU=/fit-in/600x585/filters:strip_icc():format(jpeg):mode_rgb():quality(90)/discogs-images/R-7906094-1451382934-8302.jpeg.jpg"))  
  musicList.add(ItemMusic("Watermelon Suger","Harry Styles","https://img.discogs.com/BJWFcL094VGWfMG4NyzMmTxCmAA=/fit-in/300x300/filters:strip_icc():format(jpeg):mode_rgb():quality(40)/discogs-images/R-14411523-1573993613-4770.jpeg.jpg"))  
  musicList.add(ItemMusic("Blinding Lights","The Weeknd","https://img.discogs.com/5hMesfsroK25HBnywCDJTrdyAdw=/fit-in/300x300/filters:strip_icc():format(jpeg):mode_rgb():quality(40)/discogs-images/R-14944477-1587467863-2642.jpeg.jpg"))  
  musicList.add(ItemMusic("ROXANNE","Arizona Zervas","https://img.discogs.com/uvWHDxXKUozk5kKhSfXVIeq1Mp4=/fit-in/600x600/filters:strip_icc():format(jpeg):mode_rgb():quality(90)/discogs-images/R-15678821-1595769138-5464.jpeg.jpg"))  
 }  
  
}
~~~

여기까지는 그냥 리싸이클러뷰에 아이템을 띄우는 것과 똑같다.

하지만 아이템을 클릭했을때 처리를 해주려면, 인터페이스와 추상메소드를 구현해주고 adapter에 listener를 넘겨줘야한다.

~~~
interface onClickMusic{  
  fun onClickMusic(position:Int)  
}
~~~

액티비티에 interface와 추상메소드를 적어준다. 

~~~
class MainActivity : AppCompatActivity() ,onClickMusic { ... }
~~~

그리고 만들어둔 인터페이스를 상속해서 메소드를 구현한다. 아이템이 눌렸을때 뷰에서 할 일을 적어주면 된다.

~~~
override fun onClickMusic(position: Int) {  
  val intent = Intent(this,DetailActivity::class.java)  
  intent.putExtra("music",musicList[position])  
  startActivity(intent)  
}
~~~
adapter에서 넘어온 position정보에 따라 
클릭한 아이템이 어떤 position에 있는지 알게 되고, DetailActivity에 그 정보를 넘겨야하기때문에 putExtra로 데이터를 넘겨줬다.

나같은 경우에는 어댑터의 생성자에 리스너를 넘겨줬다. 메소드를 따로 만들어서 거기에 리스너를 넘겨줘도 상관없다.
~~~
val adapter = MusicAdapter(musicList,this)
~~~

그리고 넘겨받은 리스너를 다시 viewHolder의 bind로 보내야한다. click이벤트가 발생하는 코드를 적는곳이 viewHolder의 bind이기 때문에 ㅎㅎ

마지막으로 bind함수 안에서 클릭이벤트가 발생했을때 넘겨받은 listener로 함수 매개변수로 position을 넘겨주면 된다. 

~~~
class MusicAdapter(list: ArrayList<ItemMusic>, listener: onClickMusic) : RecyclerView.Adapter<ViewHolder>() {  
  val musicList = list  
  val listener = listener
  ...
  override fun onBindViewHolder(holder: ViewHolder, position: Int) {  
  return holder.bind(musicList[position],listener)  
}
...
  }
~~~

~~~
class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView){   
  ...
  fun bind(item:ItemMusic,listener: onClickMusic){  
 ...  
  imgPhoto.setOnClickListener {  
  listener.onClickMusic(adapterPosition)  
  }  
  }  
}
~~~
