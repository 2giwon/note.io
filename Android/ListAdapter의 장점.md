구글 I/O 2018 Android Jetpack: what’s new in Android Support Library 세션에서 언급된 RecyclerView의 새로운 기능인 ListAdapter와 RecyclerView Selection에 대해서 소개합니다.

[https://www.youtube.com/watch?v=jdKUm8tGogw&feature=youtu.be&t=1137](https://www.youtube.com/watch?v=jdKUm8tGogw&feature=youtu.be&t=1137)

## ListAdapter, 더 효율적인 RecyclerView 어댑터

![https://miro.medium.com/max/648/1*4eT_cyp_kcPnkUEpw2NwPg.gif](https://miro.medium.com/max/648/1*4eT_cyp_kcPnkUEpw2NwPg.gif)

ListAdapter는 2018년 2월 서포트 라이브러리에 새로 추가된 API입니다. 

비슷한 종류로 거슬러 올라가면 RecyclerView 어댑터에서 두 리스트의 차이를 계산하는 DiffUtil이 있습니다. 

이 클래스는 약간의 보일러플레이트 코드와 두 리스트의 비교 처리를 (권고사항으로) 백그라운드 스레드에서 실행 후 결과를 메인 스레드에서 처리하는 코드가 필요했습니다. 

ListAdapter는 내부적으로 AsyncListDiffer을 사용해 개발자가 직접 DiffUtil을 사용할 때 필요했던 처리를 대신 다룹니다. 

이를 통해 더 적은 코드로 두 리스트의 차이를 계산해서 변경이 발생한 부분만 업데이트할 수 있습니다.

[DiffUtil | Android Developers](https://developer.android.com/reference/android/support/v7/util/DiffUtil)

[AsyncListDiffer | Android Developers](https://developer.android.com/reference/android/support/v7/recyclerview/extensions/AsyncListDiffer)

ListAdapter는 구글 I/O 2018의 Android Jetpack: what’s new in Android Support Library 세션과, 

Android Jetpack: manage infinite lists with RecyclerView and Paging 세션에서 

다음과 같은 특징을 갖고 있는 것으로 소개됐습니다.

[Android Jetpack: what's new in Android Support Library (Google I/O 2018)](https://youtu.be/jdKUm8tGogw?t=1137)

[https://www.youtube.com/watch?v=BE5bsyGGLf4](https://www.youtube.com/watch?v=BE5bsyGGLf4)

- 불변(Immutable) 리스트에서 동작
- Diffutil을 사용하는 간소화된 방법
- 애니메이션 업데이트 제공
- 동시성 지원

ListAdapter는 내부적으로 리스트를 읽기만 가능한 불변 객체로 다룹니다. 

따라서 전달된 리스트에서 항목을 직접 변경하는 것을 허용하지 않고, 만일 변경한다고 하더라도 업데이트는 반영되지 않습니다. 

리스트에서 항목이 수정, 추가, 삭제, 이동이 발생하는 경우, 반드시 변경이 반영된 새로운 리스트를 ListAdapter로 전달해야 합니다. 

백그라운드 스레드에서 리스트 변경 사항이 계산되면 내부적으로 notifyItem*() 함수가 호출되고, 사용자는 업데이트 된 RecyclerView를 볼 수 있습니다. 

이는 기존의 RecyclerView 어댑터와의 차이점이자 ListAdapter의 특징입니다.

이런 구조로 인해 LiveData<List> 또는 Observable<List>를 이용해 리스트의 데이터 변경을 구독하고, 변경된 리스트를 ListAdapter에 제공하는 방식으로 구현할 수 있습니다.

ListAdapter의 상태를 변경하기 위한 API는 새로운 리스트를 설정하는  

```kotlin
submitList(val list: List<T>)
```

함수가 유일합니다. 

내부 동작에 비해 외부로 노출된 인터페이스가 극도로 단순합니다. 

이를 통해 모든 복잡성을 감추면서 간단한 사용법을 제공합니다.

```kotlin
class MainActivity : AppCompatActivity() {
  private val numbersObservable: BehaviorSubject<List<Int>>
          = BehaviorSubject.createDefault(listOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9))

  override fun onCreate(savedInstanceState: Bundle?) {
    ...

    swipeRefreshLayout.setOnRefreshListener {
      numbersObservable.onNext(listOf(1, 2, 4, 5, 6, 7, 8, 9)) // pass a new list without 0, 3
    }

    numbersObservable.subscribe {
      adapter.submitList(it)
    }
  }
}
```

초기 리스트 *[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]*에서 [*0, 3]*이 삭제된 새로운 리스트를 Observable을 통해 발행하면 ListAdapter가 두 리스트에서 삭제된 항목을 찾고, notifyItemRangeRemoved()로 UI를 업데이트합니다.

ListAdapter는 두 리스트의 비교를 위해 생성자 파라미터로 DiffUtil.ItemCallback을 필요로 합니다.

```kotlin
class NumberListAdapter : ListAdapter<Int, ViewHolder>(object : DiffUtil.ItemCallback<Int>() {
  override fun areItemsTheSame(oldItem: Int?, newItem: Int?) =
      oldItem == newItem // check uniqueness

  override fun areContentsTheSame(oldItem: Int?, newItem: Int?) =
      oldItem ==  newItem // check contents
}) {

  ...                             
}
```

이쯤 되면 Paging Library의 PagedListAdapter와 꽤 유사하다고 생각할 수 있습니다. 

그런 생각이 이상하지 않은 게, 두 클래스가 내부적으로 AsyncListDiffer를 사용해서 어댑터의 핵심적인 부분을 구현하고 있기 때문입니다. 

ListView 시절에 존재했던 ListAdapter 인터페이스와 공교롭게도 이름은 같지만 관계는 없습니다.

## SelectionTracker로 RecyclerView 아이템 선택하기

![https://miro.medium.com/max/648/1*z_C7jBzKDXJgmQ6xYouo4w.gif](https://miro.medium.com/max/648/1*z_C7jBzKDXJgmQ6xYouo4w.gif)

recyclerview-selection은 2018년 3월 서포트 라이브러리 28.0.0-alpha1에서 처음 소개됐습니다. 

이 라이브러리는 RecyclerView 내에서 아이템을 선택할 수 있는 기능과 이를 관리하고 제어할 수 있는 API를 제공합니다. 

또한 구글 포토 앱에서 볼 수 있었던 드래그로 아이템을 선택하는 기능도 기본적으로 제공하며, 

다중 선택 시 터치뿐만 아니라 마우스에서도 잘 동작하도록 설계되어 있습니다.

사진을 선택하는 화면을 만든다고 가정했을 때, 개발자는 몇몇 상황을 고려해 코드를 설계해야 합니다. 

대표적인 예로 프로필 사진을 업로드하는 화면이라면 대게 단일 선택만 가능하도록 구현할 것이고, 

채팅방에서 여러 장의 사진을 공유한다면 최대 N개의 이미지만 다중 선택이 되도록 설계해야 합니다. 

또는 RecyclerView의 아이템이 여러 View 타입을 지원할 때, 

가령 특정 아이템이 폴더라면 폴더는 선택되지 않게 예외 처리가 필요합니다. 

선택 모드로 진입했을 때와 빠져나왔을 때에 어떠한 처리를 하고 싶다면 해당 시점을 알 수 있는 API가 필요할지도 모릅니다. 

recyclerview-selection 라이브러리는 바로 이러한 고민을 다룹니다.

### ItemKeyProvider<K>

추상 클래스이고, 개발자는 특정 위치에 해당하는 아이템을 식별할 수 있는 고유한 키를 제공해야 합니다. 

키 타입은 StorageStrategy와도 관련이 있는데 이 클래스는 액티비티의 onSaveInstanceState(), onRestoreInstanceState() 함수가 호출되는 시점에 선택된 키 들을 저장 했다가 복원하는 처리를 책임집니다. 

이는 화면 회전과 같은 액티비티가 재생성되는 시나리오를 recyclerview-selection 라이브러리가 내부적으로 다루고 있다는 얘기이기도 합니다. 

StorageStrategy는 기본적으로 String, Long, Parcelable을 지원하고, 만일 다른 타입의 키를 사용한다면 StorageStrategy를 상속해야 합니다.

### ItemDetailsLookup<K>

추상 클래스이고, 

개발자는 MotionEvent에 해당하는 RecyclerView 아이템을 ItemDetails 객체로 제공해야 합니다. 

이 클래스에서는 Selection hotspot이라는 개념을 소개하고, 

이를 제어할 수 있는 함수를 제공합니다. 

보통 선택 모드로 진입하기 위해서는 선택하고자 하는 항목을 롱-클릭하는 것이 일반적이지만, 

항목 내의 특정 영역을 클릭했을 때 선택 모드로 즉시 진입할 수 있는 기능을 제공할 수도 있습니다. 

이 영역을 Selection hotspot이라고 부릅니다. 

대표적인 예로 Gmail에서 제목의 왼쪽에 있는 동그란 프로필을 클릭하면 선택 모드로 진입하는 것을 볼 수 있는데, 

이런 처리가 필요한 경우 ItemDetails에서 다룰 수 있습니다.

### SelectionTracker<K>

SelectionTracker는 위에서 설명한 ItemKeyProvider, StorageStrategy, ItemDetailsLoopup, RecyclerView를 

입력으로 받으며 단일/다중 선택에 대한 설정, 상태를 알 수 있는 리스너, 사용자가 선택한 항목을 관리합니다. 

SelectionTracker는 추상 클래스이지만 Builder 클래스를 통해 DefaultSelectionTracker 구현체를 제공합니다. 

이 구현체는 사용자가 특정 항목을 선택하면 Set 자료구조에 이를 저장하고, 

어댑터의 notifyItemChanged() 함수를 호출하는 부분까지 관여합니다. 

뷰 상태를 직접 업데이트하지는 않기 때문에 개발자는 어댑터의 onBindViewHolder()에서 

선택 여부에 따른 뷰 상태를 업데이트해야 하고, 

View.setActivated() 함수를 호출해야 합니다.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
  super.onCreate(savedInstanceState)
  ...

  selectionTracker = SelectionTracker.Builder(
      "selection-demo",
      recyclerView,
      StableIdKeyProvider(recyclerView),
      itemDetailsLookup,
      StorageStrategy.createLongStorage())
    .withSelectionPredicate(selectionPredicate)
    .build()

  selectionTracker.addObserver(object : SelectionTracker.SelectionObserver<Long>() {
    override fun onSelectionChanged() {
      title = if (selectionTracker.hasSelection()) {
        "Selection ${selectionTracker.selection.size()} / $MAXIMUM_SELECTION"
      } else {
        RecyclerViewSelectionActivity::class.java.simpleName
      }
    }
  })
}
```
