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

[https://miro.medium.com/proxy/0*4ootRC3XBgBD5HQG](https://miro.medium.com/proxy/0*4ootRC3XBgBD5HQG)

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
