---
title : "코드랩 RecyclerView fundamentals(kotlin) "
excerpt : "코드랩 따라하면서 공부하기~"

categories:
  - Android
tags :
  - Kotlin
---



![스크린샷 2021-02-05 오전 1 33 41](https://user-images.githubusercontent.com/53978090/106924201-3212bc80-6752-11eb-96f9-10c1dfcd919e.png)

저번에 했던 SleepTracker 화면 RecyclerView로 만들기!

**1. xml RecyclerView로 바꾸기**

**fragment_sleep_tracker.xml**
~~~
<androidx.recyclerview.widget.RecyclerView  
  android:id="@+id/rv_sleep_list"  
  android:layout_width="0dp"  
  android:layout_height="0dp"  
  app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"  
  app:layout_constraintBottom_toTopOf="@+id/clear_button"  
  app:layout_constraintEnd_toEndOf="parent"  
  app:layout_constraintStart_toStartOf="parent"  
  app:layout_constraintTop_toBottomOf="@+id/start_button" />
~~~

화면에 RecyclerView를 설정해준다.
(여기서 layoutManager를 써주지 않으면 View에서 직접 써줘야함 ! 안쓰면 안뜸)

**2. 반복될 아이템 레이아웃 만들기**

**item_sleep_tracker.xml**
~~~
<?xml version="1.0" encoding="utf-8"?>  
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"  
  xmlns:app="http://schemas.android.com/apk/res-auto"  
  xmlns:tools="http://schemas.android.com/tools"  
  android:layout_width="match_parent"  
  android:layout_height="wrap_content">  
  
 <ImageView  android:id="@+id/img_sleep_quality"  
  android:layout_width="@dimen/icon_size"  
  android:layout_height="60dp"  
  android:layout_marginStart="16dp"  
  android:layout_marginTop="8dp"  
  android:layout_marginBottom="8dp"  
  app:layout_constraintBottom_toBottomOf="parent"  
  app:layout_constraintStart_toStartOf="parent"  
  app:layout_constraintTop_toTopOf="parent"  
  tools:srcCompat="@drawable/ic_sleep_5" />  
  
 <TextView  android:id="@+id/tv_sleep_length"  
  android:layout_width="0dp"  
  android:layout_height="20dp"  
  android:layout_marginStart="8dp"  
  android:layout_marginTop="8dp"  
  android:layout_marginEnd="16dp"  
  app:layout_constraintEnd_toEndOf="parent"  
  app:layout_constraintStart_toEndOf="@+id/img_sleep_quality"  
  app:layout_constraintTop_toTopOf="@+id/img_sleep_quality"  
  tools:text="Wednesday" />  
  
 <TextView  android:id="@+id/tv_sleep_quality"  
  android:layout_width="0dp"  
  android:layout_height="20dp"  
  android:layout_marginTop="8dp"  
  app:layout_constraintEnd_toEndOf="@+id/tv_sleep_length"  
  app:layout_constraintHorizontal_bias="0.0"  
  app:layout_constraintStart_toStartOf="@+id/tv_sleep_length"  
  app:layout_constraintTop_toBottomOf="@+id/tv_sleep_length"  
  tools:text="Excellent!!!" />  
</androidx.constraintlayout.widget.ConstraintLayout>
~~~

그다음 리싸이클러뷰를 만들때 필요한 어댑터와 뷰홀더를 설정해준다.

**Adapter**

Adapter는 반복해서 보여줘야하는 아이템 리스트1개와 3개의 메소드를 override해줘야한다.
1) onCreateViewHolder

아까 만들어둔 item_layout 을 inflater 로 inflate해서 ViewHolder로 넘겨준다.

2) onBindViewHolder

리스트[position]으로 item 한개씩 holder의 bind함수로 넘겨준다. 

3) getItemCount

리스트의 사이즈를 return 해준다.

**ViewHolder**

Adapter안에 써줘도 되고, 따로 밖에 빼도 된다. 하지만 안에 써주는게 접근할때 더 편한것 같다.

onCreateViewHolder에서 넘겨받은 레이아웃을 View로 받아
View.textview, View.imageview등 값을 설정해줄 변수들을 선언한다.

onBindViewHolder에서 bind함수로 넘겨준 item을 이용해서 위에 선언해둔 변수의 값을 설정해준다. 

**SleepNightAdpater**
~~~
class SleepNightAdapter: RecyclerView.Adapter<SleepNightAdapter.ViewHolder>(){  
  var data = listOf<SleepNight>()  
  set(value) {  
  field = value  
  notifyDataSetChanged()  
 }  override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {  
  return ViewHolder.from(parent)  
 }  
  override fun onBindViewHolder(holder: ViewHolder, position: Int) {  
  val item = data[position]  
  holder.bind(item)  
  
 }  
  override fun getItemCount(): Int {  
  return data.size  
    }  
  class ViewHolder private constructor(itemView: View) : RecyclerView.ViewHolder(itemView){  
  val tvSleepLength: TextView = itemView.findViewById(R.id.tv_sleep_length)  
  val tvSleepQuality: TextView = itemView.findViewById(R.id.tv_sleep_quality)  
  val imgSleepQuality: ImageView = itemView.findViewById(R.id.img_sleep_quality)  
  
  fun bind(item: SleepNight) {  
  val res = itemView.context.resources  
  tvSleepLength.text = convertDurationToFormatted(  
  item.startTimeMilli, item.endTimeMilli, res)  
  tvSleepQuality.text = convertNumericQualityToString(  
  item.sleepQuality, res)  
  imgSleepQuality.setImageResource(when (item.sleepQuality) {  
 0 -> R.drawable.ic_sleep_0  
  1 -> R.drawable.ic_sleep_1  
  2 -> R.drawable.ic_sleep_2  
  3 -> R.drawable.ic_sleep_3  
  4 -> R.drawable.ic_sleep_4  
  5 -> R.drawable.ic_sleep_5  
  else -> R.drawable.ic_sleep_active  
  })  
 }  
  companion object {  
  fun from(parent: ViewGroup): ViewHolder {  
  val layoutInflater = LayoutInflater.from(parent.context)  
  val view = layoutInflater  
                        .inflate(R.layout.item_sleep_tracker, parent, false)  
  return ViewHolder(view)  
 } }  
 }}
~~~

---
**참고**

코드랩 : <https://developer.android.com/codelabs/kotlin-android-training-recyclerview-fundamentals?index=..%2F..android-kotlin-fundamentals#0>
