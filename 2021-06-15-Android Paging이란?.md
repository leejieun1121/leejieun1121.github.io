---
title : "Paging이란? "
excerpt : "Android Paging라이브러리를 알아보자"

categories:
  - Android
tags :
  - Kotlin
---

> **Paging 라이브러리란??**

데이터를 페이지 단위로 UI에 표현하는 것을 도와주는 라이브러리

> 왜 쓰는가?

이전에 리스트 UI가 Top 혹은 Bottom에 도달했는지를 판단하여 이전 페이지나 다음페이지를 표현하기 위해 필요한 데이터 로딩을 요청하는 코드를 추가했어야 했다 

예를 들어, 스크롤 할 때 보여줄 정보가 더 있다면 가져오기 (load more = infinite scroll = endless scroll 이라고도 한다. )를 구현한다면 다음과 같은 과정을 거쳐야한다.

![ezgif com-gif-maker (14)](https://user-images.githubusercontent.com/53978090/121929170-e2857280-cd7b-11eb-9e2f-acd86ef41f95.gif)

**Activity**

리싸이클러뷰를 스크롤 할때마다 세가지 값을 viewModel의 listScrolled 메소드로 보낸다. 

- totalItemCount : 전체 아이템의 갯수
- visibleItemCount : 현재 화면에 보여지는 아이템 갯수
- lastVisibleItem : 현재 화면에 보여지는 아이템중 마지막에 위치한 아이템의 포지션 값

(이 방법 말고 최상단 최하단 도달 감지 방법은 여러개가 있다! canScrollHorizontally 와 canScrollVertically처럼 구글에서 만들어둔 메소드도 있다고 한다. 둘의 차이에 대해서는 좀 더 공부해서 추가 작성 해야겠다 ㅎ)

```kotlin
private fun setupScrollListener() {
        val layoutManager = binding.list.layoutManager as LinearLayoutManager
        binding.list.addOnScrollListener(object : OnScrollListener() {
            override fun onScrolled(recyclerView: RecyclerView, dx: Int, dy: Int) {
                super.onScrolled(recyclerView, dx, dy)
                val totalItemCount = layoutManager.itemCount
                val visibleItemCount = layoutManager.childCount
                val lastVisibleItem = layoutManager.findLastVisibleItemPosition()
                viewModel.listScrolled(visibleItemCount, lastVisibleItem, totalItemCount)
            }
        })
    }
```

**ViewModel**

 현재 보여지는 데이터의 갯수+마지막에 위치한 아이템의 포지션+ 5번째 항목을 보고 있음 ≥ 전체 아이템의 갯수, 새로운 데이터를 더 불러와서 보여줘야한다.

```kotlin
companion object {
        private const val VISIBLE_THRESHOLD = 5
}

fun listScrolled(visibleItemCount: Int, lastVisibleItemPosition: Int, totalItemCount: Int) {
        if (visibleItemCount + lastVisibleItemPosition + VISIBLE_THRESHOLD >= totalItemCount) {
            val immutableQuery = queryLiveData.value
            if (immutableQuery != null) {
                viewModelScope.launch {
                    repository.requestMore(immutableQuery)
                }
            }
        }
    }
```

**Repository**

똑같은 키워드로 검색을 해서 데이터를 set 해주고, page를 하나 늘려준다.

```kotlin
private const val GITHUB_STARTING_PAGE_INDEX = 1
private var lastRequestedPage = GITHUB_STARTING_PAGE_INDEX

suspend fun requestMore(query: String) {
        //검색중이면 기다려라
        if (isRequestInProgress) return
        //검색결과가 더 있다면 lastRequestedPage 추가해서 리스트 더 요청
        val successful = requestAndSaveData(query)
        if (successful) {
            lastRequestedPage++
        }
    }
```

이처럼 리스트를 더 가져오는 코드를 직접 작성하면 스크롤 할때마다 더 불러와야할지 계산해야하고, page를 직접 증가해주는 등  코드가 복잡하고 문제거리가 많았다. 

⇒ 해당 문제점들을 처리하기위해  Paging라이브러리가 등장!! 데이터를 일정한 덩어리(chunk)로 나누어 제공함으로써 성능, 메모리, 네트워크 비용을 효과적으로 다룰 수 있다! 

> **구성 요소**

- **PagedList**

    : 페이지화 된 데이터, 필요시마다 DataSource에서 페이지 정보를 가져와 PagedStorage에 저장함  

    Config 객체를 참조하여 페이징 동작 수행 

    - **Config**

        : 데이터를 어느 시점에 어느 정도 크기로 로딩할지 파악, PlaceHolder를 사용하여 계속적으로 추가 데이터를 로딩하는걸 허용할지 설정 

    - **backgroundThreadExecutor**

        : 데이터의 추가 로딩시 사용됨(최초로딩제외)

    - **mainThreadExecutor**

        : 데이터 로딩이 완료되면 콜백을 통해 UI(Adapter)로 결과를 전달

    - **boundryCallback**
        - **onZeroItemsLoaded**: 최초 로딩 시점에서 데이터 로딩 요청의 결과 데이터가 비어있는지 파악
        - **onItemAtFrontLoaded**: 일부 데이터가 로딩 된 상태에서는 최상단 데이터에 접근이 발생했는지 파악
        - **onItemAtEndLoaded**: 최하단 데이터 접근이 발생했는지 파악
    - **storage**

        : 청크 단위의 데이터들 집합 

        - **Contiguous Storage**

            : Page들 사이에 비어있는 공간이 없어 연속적으로 이어지는 chunk를 이룸, 각 Page별로 크기가 다를 수 있음

        - **Non-Contiguous Storage**

            : Page들 사이에 비어있는 공간을 허용, 각 Page마다 크기가 일정해야함 

            ex) 특정 위치로 점프(2017년도 3월로 이동) 

- **PagedListAdapter**

    (= RecyclerView.Adapter + Differ )

    : RecyclerView.Adapter를 상속, PagedList를 RecyclerView에 효율적으로 연결하기 위한 Adapter구현을 제공  

    AsyncPagedListDiffer를 통해 바인딩 요청되는 페이지와 현재 바인딩된 페이지의 비교를 비동기로 수행한 후 UI를 새로운 페이지로 업데이트함  

    page가 load될 때 callback을 수신하고, background thread에서 DiffUtil을 사용해 새로운 PagedList 의 update 내용을 계산

- **DataSource(PagingSource)**

    : 데이터를 PageList에 제공해줌  

    - **ContiguousDataSource**

        : 길이가 가변적, 페이지 로딩이 순차적으로 이루어짐

        - **ItemKeyedDataSource**

            : 페이지 데이터의 각 아이템의 고유 키를 기반으로 페이징을 수행

            ex) 이전페이지 → 제일 앞 아이템의 Key사용 / 다음페이지 → 제일 뒤 아이템의 Key를 사용해서 지정된 크기 만큼의 데이터 가져옴 

        - **PageKeyedDataSource**

            : 이전페이지키(previousKey), 다음페이지키(nextKey)를 이용하여 데이터를 가져옴

    - **PositionalDataSource**

        : 위치 기반, 길이가 정해져 있어 크기를 알 수 있고 비 순차적으로 임의의 페이지에 대한 접근이 필요할 경우에 사용함 

        - **TiledDataSource**

            : Position 기반으로 임의의 페이지에 대한 접근을 용이하게 사용하기 위함

        - **ListDataSource**

            : 메모리상에 캐시된 특정 리스트를 DataSource로 사용 

        - **LimitOffsetDataSource**

            : Room database를 사용할 경우 쿼리에 Offset과 Limit을 사용하여 일정 사이즈의 페이지 단위로 로딩할 경우 사용 

    - **RemoteMediator**

        : 네트워크 및 데이터베이스에서 페이지로 나누기 구현할 때 사용 

> **사용방법**

[https://developer.android.com/codelabs/android-paging#](https://developer.android.com/codelabs/android-paging#5)0 참고 ! 

**GithubPagingSource**

: PagingSource , 데이터를 가져옴 

 RecyclerView에서 스크롤하면 → PagingSource에서 데이터 쿼리 

- PagingSource<페이징 키의 유형, 로드된 데이터의 유형>
    - 페이징 키의 유형 : GitHub API에서 페이지 카운트 1부터 ++
    - 로드된 데이터의 유형 : Repo (깃허브 레포 데이터)
- service : 데이터 가져오는곳 (Retrofit 객체와 interface 선언)

```kotlin
private const val GITHUB_STARTING_PAGE_INDEX = 1

class GithubPagingSource(
        private val service: GithubService,//retrofit 서비스 객체 
        private val query: String //검색 키워드 
) : PagingSource<Int, Repo>() {

		//스크롤 할 때 더 많은 데이터를 비동기식으로 가져옴
    //LoadParams: 로드 작업과 관련된 정보가 저장됨(로드할 페이지의 키(초기에 null), 로드 크기(로드 요청된 항목의 수))
    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Repo> {
        //결과 : LoadResult.Page(성공)와 LoadResult.Error(실패) 로 나뉨
			val position = params.key ?: GITHUB_STARTING_PAGE_INDEX
			        val apiQuery = query + IN_QUALIFIER
			        return try {
									//retrofit 서비스를 통해 서버통신 -> 키워드로 검색한 깃허브 레포지토리 가져오기 
			            val response = service.searchRepos(apiQuery, position, params.loadSize)
			            val repos = response.items
			            val nextKey = if (repos.isEmpty()) {
			                null
			            } else {
			                // initial load size = 3 * NETWORK_PAGE_SIZE
			                // ensure we're not requesting duplicating items, at the 2nd request
			                position + (params.loadSize / NETWORK_PAGE_SIZE)
			            }
			            LoadResult.Page(
			                    data = repos,
			                    prevKey = if (position == GITHUB_STARTING_PAGE_INDEX) null else position - 1,
			                    nextKey = nextKey
			            )
			        } catch (exception: IOException) {
			            return LoadResult.Error(exception)
			        } catch (exception: HttpException) {
			            return LoadResult.Error(exception)
			        }    
			}
		
		//새로고침 키 load()함수 후에 호출됨
		//이전 키가 null이라면 next키 얻어야함, anchorPosition =  가장 최근에 접근한 인덱스
   override fun getRefreshKey(state: PagingState<Int, Repo>): Int? {
			return state.anchorPosition?.let { anchorPosition ->
			            state.closestPageToPosition(anchorPosition)?.prevKey?.plus(1)
			                ?: state.closestPageToPosition(anchorPosition)?.nextKey?.minus(1)
			        }    
		}

}
```

 

PagingData를 구성하는 방법

- Kotlin `Flow` - `Pager.flow` 사용
- `LiveData` - `Pager.liveData` 사용
- RxJava `Flowable` - `Pager.flowable` 사용
- RxJava `Observable` - `Pager.observable` 사용

PagingConfig를 사용해서 만들어둔 PagingSource에서 데이터를 어떻게 로드할지 옵션을 정함

**Repository** 

해당 프로젝트는 데이터 타입을 Kotlin Flow 사용

**pageSize** : maxSize 지정 (한 페이지당 50개씩 가져오기)

**enablePlaceholders** : true 라면, 아직 로드되지 않은 item → null표시  

```kotlin
fun getSearchResultStream(query: String): Flow<PagingData<Repo>> {
    return Pager(
          config = PagingConfig(
            pageSize = NETWORK_PAGE_SIZE,
            enablePlaceholders = false
         ),
          pagingSourceFactory = { GithubPagingSource(service, query) }
    ).flow
}

companion object {
        private const val NETWORK_PAGE_SIZE = 50
}
```

위 코드에서 넘겨받은 페이징 객체를 사용해서 viewModel 데이터에 적용 

**ViewModel**

```kotlin
class SearchRepositoriesViewModel(private val repository: GithubRepository) : ViewModel() {

    private var currentQueryValue: String? = null

    private var currentSearchResult: Flow<PagingData<Repo>>? = null

    fun searchRepo(queryString: String): Flow<PagingData<Repo>> {
        val lastResult = currentSearchResult
        if (queryString == currentQueryValue && lastResult != null) {
            return lastResult
        }
        currentQueryValue = queryString
        val newResult: Flow<PagingData<Repo>> = repository.getSearchResultStream(queryString)
                .cachedIn(viewModelScope)
        currentSearchResult = newResult
        return newResult
    }
}
```

**PagintAdapter** 설정

```kotlin
class ReposAdapter : PagingDataAdapter<Repo, RepoViewHolder>(REPO_COMPARATOR) {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RepoViewHolder {
        return RepoViewHolder.create(parent)
    }

    override fun onBindViewHolder(holder: RepoViewHolder, position: Int) {
        val repoItem = getItem(position)
        if (repoItem != null) {
            holder.bind(repoItem)
        }
    }

    companion object {
        private val REPO_COMPARATOR = object : DiffUtil.ItemCallback<Repo>() {
            override fun areItemsTheSame(oldItem: Repo, newItem: Repo): Boolean =
                oldItem.fullName == newItem.fullName

            override fun areContentsTheSame(oldItem: Repo, newItem: Repo): Boolean =
                oldItem == newItem
        }
    }
}
```

**Activity**

리싸이클러뷰에 PagingAdapter 적용

```kotlin
initAdapter() 

//만들어둔 PagingAdapter를 리싸이클러뷰에 적용 
private fun initAdapter() {
        binding.list.adapter = adapter
}
```

Flow<PagingData>를 사용하려면 새 코루틴을 실행해야함 

이전 검색 작업을 취소하고, viewModel의 searchRepo를 호출하여 결과를 pagingAdapter에 전달 

```kotlin
private var searchJob: Job? = null

private fun search(query: String) {
   // Make sure we cancel the previous job before creating a new one
   searchJob?.cancel()
   searchJob = lifecycleScope.launch {
       viewModel.searchRepo(query).collectLatest {
           adapter.submitData(it)
       }
   }
}
```

스크롤 위치 재설정 관련

PagingAdapter.loadStateFlow 사용 : 로드 상태가 변경될때마다 CombinedLoadStates 객체를 통해 메세지를 표시

- `CombinedLoadStates.refresh`: `PagingData`를 처음 로드할 때의 로드 상태를 나타냅니다.
- `CombinedLoadStates.prepend`: 목록의 시작 부분에서 데이터를 로드하는 작업의 로드 상태를 나타냅니다.
- `CombinedLoadStates.append`: 목록의 끝에서 데이터를 로드하는 작업의 로드 상태를 나타냅니다.

```kotlin
private fun initSearch(query: String) {
    ...
    // First part of the method is unchanged

        // Scroll to top when the list is refreshed from network.
        lifecycleScope.launch {
            adapter.loadStateFlow
                   // Only emit when REFRESH LoadState changes.
                   .distinctUntilChangedBy { it.refresh }
                   // Only react to cases where REFRESH completes i.e., NotLoading.
                   .filter { it.refresh is LoadState.NotLoading }
                   .collect { binding.list.scrollToPosition(0) }
        }
}
```

데이터 로딩 상태 표시

1. xml 에 progressbar 생성

```kotlin
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".ui.SearchRepositoriesActivity">
		...
		<ProgressBar
		        android:id="@+id/progress_bar"
		        style="?android:attr/progressBarStyle"
		        android:layout_width="match_parent"
		        android:layout_height="wrap_content"
		        android:layout_gravity="center"
		        app:layout_constraintLeft_toLeftOf="parent"
		        app:layout_constraintRight_toRightOf="parent"
		        app:layout_constraintTop_toTopOf="parent"
		        app:layout_constraintBottom_toBottomOf="parent"/>
		...
</androidx.constraintlayout.widget.ConstraintLayout>
```

2. 어댑터에 LoadStateListener 설정

```kotlin
adapter.addLoadStateListener { loadState ->
		binding.progressBar.isVisible = loadState.source.refresh is LoadState.Loading
}
```

> **동작과정**

발그림과 악필이지만 Paging과정을 그려보았다 ㅎ

**초기 데이터가 로드되는 과정** 

![image](https://user-images.githubusercontent.com/53978090/121929463-342dfd00-cd7c-11eb-8742-3f780e87b0dc.png)
  
**스크롤해서 새로운 데이터 로드!**

![image](https://user-images.githubusercontent.com/53978090/121929489-398b4780-cd7c-11eb-8feb-81461e63dd07.png)
  
> 결론

Paging이라는걸 알고는 있었지만.. 언젠가 해야지 하다가 공부해보자! 하고 접해봤다. 꽤나 복잡한 내부적인 흐름까지 하나하나 이해하려면 오래걸릴거같지만 page를 카운트 하지 않고 자동으로 계산해서 로드해주는 아주 좋은 기능같다 공부할게 너무 많다😅

---

[https://myungpyo.medium.com/android-paging-library-분석-ddff2662edac](https://myungpyo.medium.com/android-paging-library-%EB%B6%84%EC%84%9D-ddff2662edac)

[https://developer.android.com/codelabs/android-paging?hl=ko#1](https://developer.android.com/codelabs/android-paging?hl=ko#1)

[https://eyegochild.medium.com/android-paging-library-in-jetpack-1-overview-e40e7ce85d96](https://eyegochild.medium.com/android-paging-library-in-jetpack-1-overview-e40e7ce85d96)
