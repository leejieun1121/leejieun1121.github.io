---
title : "코드랩 Retrofit으로 데이터 가져오기3 "
excerpt : "필터기능 + 아이템 클릭시 디테일 보여주는 기능 추가하기"

categories:
  - Android
tags :
  - Kotlin
---


![ezgif com-gif-maker (13)](https://user-images.githubusercontent.com/53978090/108173454-7e420180-7141-11eb-9ae9-9d64345f3dae.gif)


### MarsProperty 값에 따라 $ set해주기 

$은 MarsProperty 객체에서 type이 rent 일때 안보이고, rent가 아닐땐 보이도록 한다.

Boolean값으로 판단해주기위해 isRental이라는 커스텀 접근자를 추가해준다.

**MarsProperty**
~~~
data class MarsProperty(  
  val id: String,  
        @Json(name = "img_src")  
  val imgSrcUrl : String,  
        val type : String,  
        val price : Double)  
 { val isRental get() = type == "rent" }
~~~

그다음 $이미지를 item 레이아웃에 추가해주고, 만들어준 [커스텀 접근자](https://umbum.dev/591)에따라 visiblity를 설정해주면 된다.

**grid_view_item.xml**

~~~
<?xml version="1.0" encoding="utf-8"?>  
  
<!--  
 ~ Copyright 2019, The Android Open Source Project ~ ~ Licensed under the Apache License, Version 2.0 (the "License"); ~ you may not use this file except in compliance with the License. ~ You may obtain a copy of the License at ~ ~     http://www.apache.org/licenses/LICENSE-2.0 ~ ~ Unless required by applicable law or agreed to in writing, software ~ distributed under the License is distributed on an "AS IS" BASIS, ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. ~ See the License for the specific language governing permissions and ~ limitations under the License. ~ -->  
<layout xmlns:android="http://schemas.android.com/apk/res/android"  
  xmlns:app="http://schemas.android.com/apk/res-auto"  
  xmlns:tools="http://schemas.android.com/tools">  
 <data> <variable  name="property"  
  type="com.example.android.marsrealestate.network.MarsProperty" />  
 <import type="android.view.View"/>  
  
 </data> <FrameLayout  android:layout_width="match_parent"  
  android:layout_height="170dp"  
  >  
  
 <ImageView  android:id="@+id/mars_image"  
  android:layout_width="match_parent"  
  android:layout_height="match_parent"  
  android:scaleType="centerCrop"  
  android:adjustViewBounds="true"  
  android:padding="2dp"  
  app:imageUrl="@{property.imgSrcUrl}"  
  tools:src="@tools:sample/backgrounds/scenic"/>  
  
 <ImageView  android:id="@+id/mars_property_type"  
  android:layout_width="wrap_content"  
  android:layout_height="45dp"  
  android:layout_gravity="bottom|end"  
  android:adjustViewBounds="true"  
  android:padding="5dp"  
  android:scaleType="fitCenter"  
  android:src="@drawable/ic_for_sale_outline"  
  android:visibility="@{property.rental ? View.GONE : View.VISIBLE}"  
  tools:src="@drawable/ic_for_sale_outline"  
  />  
  
 </FrameLayout>  
  
</layout>
~~~

---
### 필터에 따른 리스트 설정 

> 필터 선택 메뉴 만들기

오른쪽 상단 위 버튼을 누르면 필터를 선택할 수 있는 메뉴를 만든다.

res->menu

**overflow_menu.xml**

~~~
<?xml version="1.0" encoding="utf-8"?>  
  
<!--  
 ~ Copyright 2019, The Android Open Source Project ~ ~ Licensed under the Apache License, Version 2.0 (the "License"); ~ you may not use this file except in compliance with the License. ~ You may obtain a copy of the License at ~ ~     http://www.apache.org/licenses/LICENSE-2.0 ~ ~ Unless required by applicable law or agreed to in writing, software ~ distributed under the License is distributed on an "AS IS" BASIS, ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. ~ See the License for the specific language governing permissions and ~ limitations under the License. ~ -->  
<menu xmlns:android="http://schemas.android.com/apk/res/android">  
 <item  android:id="@+id/show_all_menu"  
  android:title="@string/show_all" />  
 <item  android:id="@+id/show_rent_menu"  
  android:title="@string/show_rent" />  
 <item  android:id="@+id/show_buy_menu"  
  android:title="@string/show_buy" />  
</menu>
~~~

> 만든 메뉴 프레그먼트에 보이도록 하기

**OverviewFragment.kt**

~~~
override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,  
                          savedInstanceState: Bundle?): View? {
                          ...
			setHasOptionsMenu(true)
		...
}
~~~

~~~
override fun onCreateOptionsMenu(menu: Menu, inflater: MenuInflater) {  
  inflater.inflate(R.menu.overflow_menu, menu)  
  super.onCreateOptionsMenu(menu, inflater)  
}
~~~

> API추가하기

Show all/ Rent / Buy 라는 filter 에 따라 각각 다른 MarsProperty리스트가 response로 온다.
filter를 보내기 위해서는 @Query를 사용해야함 

~~~
interface MarsApiService{  
  @GET("realestate")  
  suspend fun getProperties  
  (@Query("filter") type: String)  
  : List<MarsProperty>  
  
}
~~~

> filter enum class 로 지정

구분하기 쉽게 3가지 필터를 enum class 로 만들어준다.

~~~
enum class MarsApiFilter(val value:String)  
{  
	SHOW_RENT("rent"),  
    SHOW_BUY("buy"),  
    SHOW_ALL("all")  
  
}
~~~

> 바뀐 API 사용해서 통신하기

먼저 어플 실행시켰을때 처음의 상태를 Show All으로 지정해준다.

**OverviewViewModel.kt**
~~~
init {  
  getMarsRealEstateProperties(MarsApiFilter.SHOW_ALL)  
  
}

private fun getMarsRealEstateProperties(filter:MarsApiFilter){  
  viewModelScope.launch {  
  _properties.value = MarsApi.retrofitService.getProperties(filter.value)  
  }  
}
~~~

> 메뉴 선택했을때 바뀐 리스트 적용시키기

옵션메뉴를 선택했을때 선택한 메뉴에 따라 Filter를 다르게 해서 호출을 해줘야한다.

~~~
override fun onOptionsItemSelected(item: MenuItem): Boolean {  
  viewModel.updateFilter(  
  when (item.itemId) {  
  R.id.show_rent_menu -> MarsApiFilter.SHOW_RENT  
  R.id.show_buy_menu -> MarsApiFilter.SHOW_BUY  
  else -> MarsApiFilter.SHOW_ALL  
  }  
 )  return true  
}
~~~

viewModel에서 업데이트 해주는 함수를 만든다.
 
~~~
fun updateFilter(filter: MarsApiFilter) {  
  getMarsRealEstateProperties(filter)  
}
~~~ 

---

### 아이템 클릭시 Detail정보 넘겨 보여주기 

아이템 클릭을 구현하기 위해 인터페이스와 추상메소드를 선언해준다.

**OverviewFragment.xml**

~~~
interface OnItemClickListener{  
  fun onItemClick(item : MarsProperty)  
}
~~~

그다음 Fragment에서 구현한다. 그리고 Adapter에 리스너를 넘겨줘서 클릭했을때 Callback해줘야한다.

클릭한 item을 DetailFragment 에서 보여줄거기 때문에viewModel.displayPropertyDetails의 인수로 넘긴다.

~~~
class OverviewFragment : Fragment(),OnItemClickListener {
...
override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,  
                          savedInstanceState: Bundle?): View? {
                          ...
binding.rvPhotoList.adapter = PhotoGridAdapter(this)
...
}


override fun onItemClick(item: MarsProperty) {  
  viewModel.displayPropertyDetails(item)  
}

}
~~~

Adapter로 넘겨받은 listener를 이용해서 아이템이 눌렸을때의 해당 포지션 marsProperty객체를 가져올 수 있다.

**PhotoGridAdapter.kt**

~~~
class PhotoGridAdapter(private val onClickListener:OnItemClickListener) : ListAdapter<MarsProperty, PhotoGridAdapter.MarsPropertyViewHolder>(DiffCallback){

...
override fun onBindViewHolder(holder: MarsPropertyViewHolder, position: Int) {  
  val marsProperty = getItem(position)  
  holder.itemView.setOnClickListener {  
  onClickListener.onItemClick(marsProperty)  
  }  
  holder.bind(marsProperty)  
}
...


}
~~~

아까 viewModel의 displayPropertyDetails 함수의 인수로 클릭된 position의 MarsProperty객체를 넘겨줬다.

displayPropertyDetails 함수에서는 _navigateToSelectedProperty에 넘어온 객체를 set해주고, Fragment에서 해당 LiveData를 관찰하고 있다가 set되면 그 객체를 DetailFragment로 넘긴다.

**OverviewViewModel**

~~~
private val _navigateToSelectedProperty = MutableLiveData<MarsProperty>()  
val navigateToSelectedProperty: LiveData<MarsProperty>  
  get() = _navigateToSelectedProperty

...

fun displayPropertyDetails(marsProperty: MarsProperty) {  
  _navigateToSelectedProperty.value = marsProperty  
}  
  
fun displayPropertyDetailsComplete() {  
  _navigateToSelectedProperty.value = null  
}
~~~

**OverviewFragment.kt**

~~~
viewModel.navigateToSelectedProperty.observe(viewLifecycleOwner, Observer {  
  if(null!=it){  
  this.findNavController().navigate(OverviewFragmentDirections.actionShowDetail(it))  
  viewModel.displayPropertyDetailsComplete()  
 }})
~~~

(이때 Navigation에서 OverviewFragment -> DetailFragment로 이어져있어야한다.)

그리고 DetailFragment에서 MarsProperty라는 객체를 받는거기 때문에, MarsProperty에 @Parcelize를 붙여줘야한다.

**MarsProperty**

~~~
@Parcelize  
data class MarsProperty(  
  val id: String,  
        @Json(name = "img_src")  
  val imgSrcUrl : String,  
        val type : String,  
        val price : Double)  
  : Parcelable { val isRental get() = type == "rent" }
~~~

**nav_graph.xml**

~~~
<argument  
  android:name="selectedProperty"  
  app:argType="com.example.android.marsrealestate.network.MarsProperty"/>
~~~

마지막으로 DetailFragment에서 넘겨받은 MarsProperty를 레이아웃에 set해주면 된다.

**fragment_detail.xml**

~~~
<?xml version="1.0" encoding="utf-8"?>  
  
<!--  
 ~ Copyright 2019, The Android Open Source Project ~ ~ Licensed under the Apache License, Version 2.0 (the "License"); ~ you may not use this file except in compliance with the License. ~ You may obtain a copy of the License at ~ ~     http://www.apache.org/licenses/LICENSE-2.0 ~ ~ Unless required by applicable law or agreed to in writing, software ~ distributed under the License is distributed on an "AS IS" BASIS, ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. ~ See the License for the specific language governing permissions and ~ limitations under the License. ~ -->  
<layout xmlns:android="http://schemas.android.com/apk/res/android"  
  xmlns:app="http://schemas.android.com/apk/res-auto"  
  xmlns:tools="http://schemas.android.com/tools">  
  
 <data> <variable  name="viewModel"  
  type="com.example.android.marsrealestate.detail.DetailViewModel" />  
 </data>  
 <ScrollView  android:layout_width="match_parent"  
  android:layout_height="match_parent"  
  tools:context=".DetailFragment">  
  
 <androidx.constraintlayout.widget.ConstraintLayout  android:layout_width="match_parent"  
  android:layout_height="wrap_content"  
  android:padding="16dp">  
  
 <ImageView  android:id="@+id/main_photo_image"  
  android:layout_width="0dp"  
  android:layout_height="266dp"  
  android:scaleType="centerCrop"  
  imageUrl="@{viewModel.selectedProperty.imgSrcUrl}"  
  app:layout_constraintEnd_toEndOf="parent"  
  app:layout_constraintStart_toStartOf="parent"  
  app:layout_constraintTop_toTopOf="parent"  
  tools:src="@tools:sample/backgrounds/scenic" />  
  
 <TextView  android:id="@+id/property_type_text"  
  android:layout_width="wrap_content"  
  android:layout_height="wrap_content"  
  android:layout_marginTop="16dp"  
  android:textColor="#de000000"  
  displayPropertyType="@{viewModel.selectedProperty}"  
  android:textSize="39sp"  
  app:layout_constraintStart_toStartOf="parent"  
  app:layout_constraintTop_toBottomOf="@+id/main_photo_image"  
  tools:text="To Rent" />  
  
 <TextView  android:id="@+id/price_value_text"  
  android:layout_width="wrap_content"  
  android:layout_height="wrap_content"  
  android:layout_marginTop="8dp"  
  android:textColor="#de000000"  
  displayPropertyPrice="@{viewModel.selectedProperty}"  
  android:textSize="20sp"  
  app:layout_constraintStart_toStartOf="parent"  
  app:layout_constraintTop_toBottomOf="@+id/property_type_text"  
  tools:text="$100,000" />  
  
 </androidx.constraintlayout.widget.ConstraintLayout> </ScrollView></layout>
~~~

**DetailViewModelFactory**

~~~
class DetailViewModelFactory(  
  private val marsProperty: MarsProperty,  
        private val application: Application) : ViewModelProvider.Factory {  
  @Suppress("unchecked_cast")  
  override fun <T : ViewModel?> create(modelClass: Class<T>): T {  
  if (modelClass.isAssignableFrom(DetailViewModel::class.java)) {  
  return DetailViewModel(marsProperty, application) as T  
  }  
  throw IllegalArgumentException("Unknown ViewModel class")  
 }}
~~~

**DetailViewModel.kt**

~~~
//AndroidViewModel과 ViewModel의 차이점  
//AndroidViewModel은 컨택스트를 제공함 static 인스턴스를 가지면 이 뷰모델 사용하는게 더 나음 !class DetailViewModel(marsProperty: MarsProperty, app: Application) : AndroidViewModel(app) {  
  private val _selectedProperty= MutableLiveData<MarsProperty>()  
  val selectedProperty : LiveData<MarsProperty>  
  get() = _selectedProperty  
  
    init {  
  _selectedProperty.value = marsProperty  
  }  
}
~~~

**DetaiFragment.kt**

~~~
class DetailFragment : Fragment() {  
  override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,  
                              savedInstanceState: Bundle?): View? {  
  
  @Suppress("UNUSED_VARIABLE")  
  val application = requireNotNull(activity).application  
  val marsProperty = DetailFragmentArgs.fromBundle(arguments!!).selectedProperty  
  val viewModelFactory = DetailViewModelFactory(marsProperty,application)  
  
  val binding = FragmentDetailBinding.inflate(inflater)  
  binding.viewModel = ViewModelProvider(this, viewModelFactory).get(DetailViewModel::class.java)  
  binding.lifecycleOwner = this  
 return binding.root  
  }  
}
~~~

**BindingAdapter**

~~~
@BindingAdapter("displayPropertyPrice")  
fun displayPropertyPrice(textView :TextView, item:MarsProperty){  
  when(item.isRental){  
  true -> textView.text = "${item.price/12}" + ""  
  false -> textView.text = "${item.price},.0f"  
  }  
}  
  
@BindingAdapter("displayPropertyType")  
fun displayPropertyType(textView :TextView, item:MarsProperty){  
  when(item.isRental){  
  true -> textView.text = "For Rent"  
  false -> textView.text = "For Sale"  
  }  
}
~~~

---

**참고**

https://developer.android.com/codelabs/kotlin-android-training-internet-filtering?index=..%2F..android-kotlin-fundamentals#0
