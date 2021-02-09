---
title : "DiffUtil 및 RecyclerView를 사용한 데이터 바인딩 "
excerpt : "코드랩 RecyclerView 2번째 "

categories:
  - Android
tags :
  - Kotlin
---


**DiffUtil사용하는 이유**

-> 데이터가 바뀔때마다 notifyDataSetChanged()를 사용해줘서 전체 리스트를 변경하는 번거로움이 있었다.

**DiffUtil** 이라는 클래스는 두 목록간의 차이점을 찾고 업데이트 되어야 할 목록을 반환해주며 최소한의 업데이트 수를 계산함 !

그래서 해결책으로 DiffUtil을 사용한다고 한다. :blush: 

**1. DiffUtillCallback 클래스 구현하기**

DiffUtil.ItemCallback을 상속하는 클래스를 구현해준다. <>안에 반환해줄 자료형을 써주기 

- areItemsTheSame 
: 전달된 두 항목 oldItem, newItem이 동일한지 체크해서(id비교) Boolean값 리턴 

- areContentsTheSame
: 내부에서 동일한 데이터가 있는지 확인, 모든 필드 검사  

**SleepNightDiffCallback.kt**

~~~
package com.example.android.trackmysleepquality.sleeptracker  
  
import androidx.recyclerview.widget.DiffUtil  
import com.example.android.trackmysleepquality.database.SleepNight  
  
class SleepNightDiffCallback : DiffUtil.ItemCallback<SleepNight>() {  
  override fun areItemsTheSame(oldItem: SleepNight, newItem: SleepNight): Boolean {  
  return oldItem.nightId == newItem.nightId  
    }  
  
  override fun areContentsTheSame(oldItem: SleepNight, newItem: SleepNight): Boolean {  
  return oldItem == newItem  
  }  
}
~~~

**2. RecyclerView Adapter 바꿔주기**

RecyclerView.Adapter<SleepNightAdpater.ViewHolder>를 상속하던 어댑터를 ListAdapter<데이터클래스, SleepNightAdapter.ViewHolder>(콜백)을 상속하도록 바꿔주고 ListAdapter의 메소드인 getItem을 onBindViewHolder에 사용해줍니다.

**SleepNightAdapter**
~~~
class SleepNightAdapter: ListAdapter<SleepNight, SleepNightAdapter.ViewHolder>(SleepNightDiffCallback()){  
  override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {  
  return ViewHolder.from(parent)  
 }  
  override fun onBindViewHolder(holder: ViewHolder, position: Int) {  
  val item = getItem(position)  
  holder.bind(item)  
 }
 }
~~~

**3. Item데이터바인딩 해주기**

item_sleep_tracker에서 데이터바인딩을 사용하기 위해 layout과 sleepNight태그를 추가해줘야한다. 

**item_sleep_tracker.xml**
~~~
<layout  
  xmlns:android="http://schemas.android.com/apk/res/android"  
  xmlns:app="http://schemas.android.com/apk/res-auto"  
  xmlns:tools="http://schemas.android.com/tools"  
  >  
  
 <data> <variable  name="sleep"  
  type="com.example.android.trackmysleepquality.database.SleepNight" />  
 </data>
 ...
 </layout>
~~~

그리고 viewHolder에서 view로 inflate했던 부분을 데이터바인딩으로 inflate하도록 바꿔준다.

~~~
companion object {  
  fun from(parent: ViewGroup): ViewHolder {  
  val layoutInflater = LayoutInflater.from(parent.context)  
  val binding = ItemSleepTrackerBinding.inflate(layoutInflater,parent,false)  
  return ViewHolder(binding)  
 }}
~~~

viewHolder의 인수로 binding 을 넘겼기 때문에, constructor의 매개변수도 ItemSleepTrackerBinding으로 바꿔줘야한다.

~~~
class ViewHolder private constructor(val binding: ItemSleepTrackerBinding) : RecyclerView.ViewHolder(binding.root)
~~~

그리고 데이터바인딩을 사용했기 때문에ViewHolder에서 view.findViewById로 찾아줬던 변수들을 삭제해줘도 된다. 

**4. 바인딩 어댑터 사용하기**

저번에 Transformation을 사용해서 바인딩하는 데이터를 바꿔줬었는데, 이번엔 바인딩 어댑터를 사용한다. 바꿀 내용이 간단할때는 Transformations을, 보다 복잡한 내용은 바인딩 어댑터를 사용하는것이 좋다.  :thumbsup: 

바인딩 어댑터를 선언할 BindingUtils.kt를 만들어주고 그 안에 함수를 만들어주면 된다.

그리고 viewHolder에서 TextView나 ImageView의 값을 설정해줄때 썼던 내용 그대로 써주면 끝 ㅎㅎ


**BindingUtils.kt**
~~~
package com.example.android.trackmysleepquality.sleeptracker  
  
import android.widget.ImageView  
import android.widget.TextView  
import androidx.databinding.BindingAdapter  
import com.example.android.trackmysleepquality.R  
import com.example.android.trackmysleepquality.convertDurationToFormatted  
import com.example.android.trackmysleepquality.convertNumericQualityToString  
import com.example.android.trackmysleepquality.database.SleepNight  
  
  @BindingAdapter("sleepDurationFormatted")  
  fun TextView.setSleepDurationFormatted(item: SleepNight){  
  text = convertDurationToFormatted(item.startTimeMilli,item.endTimeMilli,context.resources)  
 }  
  @BindingAdapter("sleepQualityString")  
  fun TextView.setSleepQualityString(item: SleepNight) {  
  text = convertNumericQualityToString(item.sleepQuality, context.resources)  
 }  
  @BindingAdapter("sleepImage")  
  fun ImageView.setSleepImage(item: SleepNight) {  
  setImageResource(when (item.sleepQuality) {  
 0 -> R.drawable.ic_sleep_0  
  1 -> R.drawable.ic_sleep_1  
  2 -> R.drawable.ic_sleep_2  
  3 -> R.drawable.ic_sleep_3  
  4 -> R.drawable.ic_sleep_4  
  5 -> R.drawable.ic_sleep_5  
  else -> R.drawable.ic_sleep_active  
  })  
 }
~~~

바인딩 어댑터 ("이름")을 사용해서, xml안의 app:이름 ~ 이렇게 접근할 수 있다. 

**item_sleep_tracker.xml**

~~~
<?xml version="1.0" encoding="utf-8"?>  
<layout  
  xmlns:android="http://schemas.android.com/apk/res/android"  
  xmlns:app="http://schemas.android.com/apk/res-auto"  
  xmlns:tools="http://schemas.android.com/tools"  
  >  
  
 <data> <variable  name="sleep"  
  type="com.example.android.trackmysleepquality.database.SleepNight" />  
 </data>  
<androidx.constraintlayout.widget.ConstraintLayout  
  android:layout_width="match_parent"  
  android:layout_height="wrap_content">  
  
 <ImageView  android:id="@+id/img_sleep_quality"  
  android:layout_width="@dimen/icon_size"  
  android:layout_height="60dp"  
  android:layout_marginStart="16dp"  
  android:layout_marginTop="8dp"  
  app:sleepImage="@{sleep}"  
  android:layout_marginBottom="8dp"  
  app:layout_constraintBottom_toBottomOf="parent"  
  app:layout_constraintStart_toStartOf="parent"  
  app:layout_constraintTop_toTopOf="parent"  
  tools:srcCompat="@drawable/ic_sleep_5" />  
  
 <TextView  android:id="@+id/tv_sleep_length"  
  android:layout_width="0dp"  
  android:layout_height="20dp"  
  app:sleepDurationFormatted="@{sleep}"  
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
  app:sleepQualityString="@{sleep}"  
  app:layout_constraintEnd_toEndOf="@+id/tv_sleep_length"  
  app:layout_constraintHorizontal_bias="0.0"  
  app:layout_constraintStart_toStartOf="@+id/tv_sleep_length"  
  app:layout_constraintTop_toBottomOf="@+id/tv_sleep_length"  
  tools:text="Excellent!!!" />  
</androidx.constraintlayout.widget.ConstraintLayout>  
</layout>
~~~

마지막으로 Fragment에서 adapter.submitList()를 사용한 코드로 바꿔주면 끝 

**SleepTrackerFragment.kt**

~~~
sleepTrackerViewModel.nights.observe(viewLifecycleOwner, Observer {  
  it?.let{  
  adapter.submitList(it)  
  }  
})
~~~

---

**참고**

<https://developer.android.com/codelabs/kotlin-android-training-diffutil-databinding/#0>
