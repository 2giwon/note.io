# Intro

- Recyclerview는 가장 많이 사용하는 위젯
- 올바르게 사용안하면 성능 저하 될 수 있음
- Recyclerview가 view에서 inflation을 검색하는 방법 소개

# 어떻게 동작하나?

- 가장 중요한 메소드는 `getViewByPosition()`
- 이 메소드를 연구하면 ViewHolder recycle, 숨겨진 View 등의 개념을 잘 이해할 수 있음

LayoutManager가 뷰를 inflate하면 다음단계를 진행:

1. RecyclerView로 이동하여 해당 위치에 대한 view를 요청
2. 응답에서 RecyclerView는 Pool 또는 다른 위치에서 view를 검색
3. 적절한 view가 없으면 onCreateViewHolder를 사용하여 새 view를 생성
4. 그 후 onBindViewHolder를 통해 view를 바인딩

RecyclerView가 View를 검색하는 위치

1. RecyclerViewPool
2. View Cache
3. ViewCacheExtension
4. Hidden Views
5. 변경 및 첨부된 Scrap Views
6. stable IDs가 있는 View

## RecycledViewPool

- 재활용 된 모든 View가 저장되는 장소를 Pool이라고 함
- 화면을 아래로 스크롤 하면 위에서 본 View를 재활용
- 재활용할 View는 Pool로 이동하여 바닥에서 오는 View를 그리는데 재사용
- Recyclerview가 inflate할 ViewHolder를 검색하려면 View Type이 중요함
- 각 View Type은 고유한 용량이 있음.
- 이 용량은 다음과 같은 방법으로 수정 가능

```kotlin
recyclerView.getRecycledViewPool().setMaxRecycledViews(SOME_VIEW_TYPE, POOL_CAPACITY);
```

이 기능의 성능향상 방법

### Case 1 : list에서 유사한 View가 포함된 경우

- Pool용량을 늘릴 수 있음
- 뷰의 재활용과 성능을 증가

### Case 2: list에 다른 View가 포함된 경우

- Pool용량을 줄일 수 있음
- 메모리 공간 확보에 용이

재사용을 위해서는 ViewHolders를 Pool로 던져야함.

ViewHolders를 Pool에 던지기위한 적합한 시기를 결정하는 요소가 있음.

1. View Cache의 항목이 업데이트 되거나 제거됨.
2. Id및 View Type이 제공된 View와 일치하지 않은경우
3. 스크롤 하는 동안 View가 RecyclerView 경계를 벗어나는 경우
4. View의 데이터가 변경된 경우
5. LayoutManager는 Pre-Layout에 View를 추가했지만 post layout에 View를 포함하지 않음.

### ViewCache:

- ViewHolder는 위치와 함께 View를 매핑함.
- View를 그대로 재사용 할 수 있음.

### ViewPool:

- ViewHolder는 속성을 유지하지 않고 View와 View Type 간의 관계를 유지
- 항목이 업데이트 되거나 제거되면 해당 VIewHoder를 Pool에 던지는 위의 번호 1번에 해당
- 2번 에서도 동일한 프로세스가 진행됨.

### 3번 4번:

- View가 뷰영역을 벗어날때마다 ViewHoder를 Pool에 던져서 다른 View에 재사용할 수 있도록 함.
- Pool은 ViewType으로 View를 매핑
- 동일한 ViewType을 유지하면서 View의 데이터가 변경되면 View가 Pool에 들어갈 수 있음.

### Pre-Layout, Post-Layout

2개의 뷰 A + B 가 있는경우

![https://miro.medium.com/proxy/0*qvfwWyg_PAePcTuG.png](https://miro.medium.com/proxy/0*qvfwWyg_PAePcTuG.png)

 

- View B를 삭제
- View C를 추가
- View C가 View B 아래에 삽입된다는 것은 잘못된 가정
- Custom LayoutManager 경우 안됨
- RecyclerView는 LayoutManager에 두 가지 유형의 레이아웃을 요청
- 첫번째는 Pre-Layout
- 두번째는 Post-Layout

![https://miro.medium.com/proxy/0*4ootRC3XBgBD5HQG](https://miro.medium.com/proxy/0*4ootRC3XBgBD5HQG)

- View B 가 삭제되고 View C가 그 자리를 차지
- 이 경우 Post-Layout이 그림 처럼 나타나 애니메이션을 예측
- 이 애니메이션을 Predictive Animation
- 추가 되는 View가 Pre-Layout 이나 Post-Layout 에 없음

View B를 변경했다면?

동일하게 동작하지만 View C가 Pool 로 이동.

> *이러한 모든 조건에도 불구하고 RecyclerView에서 View를 사용할 수 있도록 하기위해서는 더 많은 장애물이 있다.*

### 1. 재활용성

- View를 RecyclerView에 사용할 수 있는지에 대한 여부를 확인하는 Flag
- setIsRecyclable() 메소드를 사용하여 이 Flag를 true로 설정
- 쌍으로 동작
- true, true 두번 하면 false, false 두번 설정

### 2. 과도한 상태

- View를 재활용 할 수 있는지 여부를 확인하는 데 도움이 되는 것
- 애니메이션을 시작할때마다 이 flag를 true로 설정
- 애니메이션이 끝나면 flag를 false로 설정

> *이러한 장애물을 넘은 후에 RecyclerView에서 ViewHolder를 사용할 수 있게 됩니다. 
또한 ViewHolder는 ViewType에 따라 결정되는 각 Pool로 이동합니다.

Cache는 Position을 처리하는 반면 Pool은 ViewType을 처리하는 것을 잊지마세요.*

# View Cache

ViewCache 는 Cache 라고도 합니다.

Cache의 기본 용량은 2입니다.

Cache에서 View를 찾으면 View초기화 하거나 바인딩 할 필요가 없으며 있는 그대로 inflate됩니다.

### **요약:**

→ View가 어디에도 없으면 새 View가 picture에 나타남

→ View가 Pool에 있는 경우 바인딩 됨

→ View가 Cache에 있으면 생성이나 Bounding이 필요하지 않음

따라서 View가 보이는 화면 경계를 벗어나면 언제든지 Cache또는 Pool로 이동할 수 있음.

또한 View를 검색하는 동안 Cache또는 Pool에서 나타날 수 있음.

주어진 시점에서 View를 추적하기 위해 어댑터의 두가지 방법을 사용할 수 있음.

### 위의 것은 :

1. onViewAttachedToWindow()
2. onDetachFromWindow()

View를 Cache로 던지는 방법도 있음.

> Cache가 비어있고 기본 용량이 있는 Pool이 있는경우 Cache가 
가득차지 않는 한 ViewHolder는 Cache로 이동함.
Cache가 가득 차면 풀로 들어가기 시작함.

화면을 위아래로 스크롤할 때 일부 동작

![https://miro.medium.com/proxy/0*cUiKS4_r-DwP1t4s](https://miro.medium.com/proxy/0*cUiKS4_r-DwP1t4s)

## Scroll Downwards

- 아래로 스크롤하면 Cache항목과 Pool로 구성된 뷰의 꼬리가 있음.
- 위 그림에서 View 8 이 화면에 표시되기 시작하고
- ViewHolder가 Cache에 있으면 Pool View를 사용.
- View 6이 상단에 사라지기 시작하면 View 4를 Pool로 push
- Cache 로 이동

## Scroll Upwards

- 위 스크롤은 다름
- View가 먼저 Cache에서 시작
- 바인딩이 필요하지 않으므로 로딩시작이 효율적

# ViewCacheExtension

- 사용자가 직접 설정 한 특수한 유형의 캐시
- 구현에 따라 동작
- 설정하려면 RecyclerView의 setViewCacheExtension() 메소드를 사용한 다음 인터페이스를 구현

### ViewCacheExtension의 사용법

- ViewCacheExtension 의 주된 효과적인 사용은 고정된 위치, 수량이 있는 ViewHolder가 있을때
- 이러한 종류의 View가 반복해서 바인딩 되는 것을 방지
- ViewType을 신경쓰지 않으므로 캐시를 사용할 수 없음
- 이런 경우 사용하면 좋음
- 제대로 사용하지 않으면 문제가 발생

구현할 인터페이스

```java
View getViewForPositionAndType(Recycler recycler, int position,int type);
```

- View뿐만 아니라 해당 ViewHolder에 해당하는 View를 반환해야 됨
- 로직은 ViewHolder를 만들 때마다 참조가 View의 레이아웃 매개 변수에 저장되는 것
- RecyclerView에 View를 요청하면 RecyclerView는 ViewCacheExtension에서 해당 View를 찾음
- ViewCacheExtension의 문제는 ViewHolder의 다른 상태가 있다는 것
- 플래그와 위치도 포함
- RecyclerView는 모든 것을 처리

## 구현 중 주의사항

ViewCacheExtension 메서드를 사용하여 View를 얻고 해당 View가 전달된 위치에 대한
View와 일치하지 않으면 버그가 발생합니다. 

## 무엇이 잘못되었나?

1. LayoutManager는 먼저 AdapterHelper 에게 마지막 레이아웃 이후의 모든 어댑터 변경 사항을 처리하도록 요청
2. LayoutManager는 RecyclerView를 호출하여 ViewHolders의 위치에 오프셋을 적용
3. 이에 대한 응답으로 RecyclerView는 현재 표시된 View의 ViewHolders를 반복

이것이 문제의원인!

- ViewCacheExtension을 사용하여 캐시 한 일부 View가 있다고 가정
- RecyclerView가 이러한 View의 오프셋을 적용하지 못하면 충돌 발생!
- 이에 대한 응답으로 RecyclerView는 현재 표시된 View의 ViewHolders를 반복

이것이 문제가 발생하는 지점.

ViewCacheExtension 을 사용하여 캐시 한 일부 View가 있다고 가정

Recyclerview가 이러한 뷰에 오프셋을 적용하지 못하면 충돌 발생

# Hidden Views

- RecyclerView 바운드에 벗어나지만 자식으로 남아 있는 View
- View를 경계밖으로 던지는 과정을 애니메이션 하는 데 사용
- LayoutManager의 관점에서 이러한 View는 Gone됩니다.
- View를 검색하는 동안에는 계산에 포함되지 않음

숨겨진 view가 그림에 나타나는 상황을 고려해보면 : 

1. 두 개의 뷰 A와 B
2. View B를 삭제하고 View C를 삽입
3. 사라지는 애니메이션이 끝나기 전에 View B가 다시 필요하다고 결정.

RecyclerView는 ChildHelper 로 이동

> *ChildHelper는 Hidden Views 및 기타 View의 인덱스를 관리*

- ChildHelper 에서 View B를 다시 가져옴.
- Hidden View를 사용하여 View를 다시 가져옴.

# 변경 및 첨부된 Scrap Views

## Scrap

- RecyclerView가 ViewHolder를 검색하는동안 가장 먼저 검색하는 뷰 목록
- LayoutManager 가 View 배치를 시작하면 모든 View가 스크랩에 덤프
- 스크랩된 곳에서 View가 하나씩 선택되고 inflate 됩니다.

예를 들면:

- View A, B, C, D 가 있고
- A, B, C 가 visible한 상태라면

1. View B 를 삭제한다면
2. 다시 re-Layout이 발생하고 모든 View가 Scrap에 덤프
3. 그 후 View A와 C는 스크랩에서 가져오고 View D는 Pool 또는 Cache에서 가져옴

View B가 Post-layout 이후에 추가되지 않았으므로 View B는 재사용을 위해 Pool로 이동

![https://miro.medium.com/proxy/0*uVqR712O-WUg88lb.png](https://miro.medium.com/proxy/0*uVqR712O-WUg88lb.png)

> *Scrap은 또한 LayoutManager와 RecyclerView의 기능이 서로 겹치지 않도록 합니다.*

RecyclerView의 Scrap 유형

1. Attached Scrap : Pre-Layout에서만 사용 가능
2. Changed Scrap : Pre 또는 Post layout중에만 사용할 수 있음

ViewHolder의 관련항목이 변경되고

ItemAnomator가 cross-fade 애니메이션과 같은 일부 애니메이션을 적용하려는 경우 View가 변경된 

스크랩으로 이동합니다.

그렇지 않은 경우  케이스 보기는 첨부된 Scrap에 덤프됩니다.
