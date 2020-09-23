# Intro

액션바에서 액션 모드로 전환하고 액션 모드에서 액션바로 전환 시

다시 액션모드에서 액션바로 전환시 

액션 바가 사라지면서 전체 뷰 크기가 다시 조정되고

액션 모드 바가 생기면서 다시 뷰의 크기가 조정되기 때문에

덜커덕 하는 문제가 발생할 수 있다.

![image](https://github.com/2giwon/note.io/blob/master/TroubleShooting/resource/actionbar_2.jpg)

![image](https://github.com/2giwon/note.io/blob/master/TroubleShooting/resource/actionbar.jpg)

인터넷 검색을 할때에는

ActionBar show hide flickering 

이렇게 검색하면 된다.

# 해결책

해결하는 방법은 AppTheme Style에서 아래와 같은 코드를 넣으면 된다.

```kotlin
<item name="android:windowActionModeOverlay">true</item>
<item name="windowActionModeOverlay">true</item>
```

액션모드 윈도우를 Overlay 로 그린다는 뜻이다.

위의 코드로 적용해도 왠만한 문제는 없을 것이다.

그러나 다른문제가 있을 수 있다.

이것은 일반적인 상황에서는 발생하지 않는다.

몇몇 사람들 중에는 내가 사용하는 액티비티 에서만 사용하고 싶을때가 있다.

AppTheme에 적용을 하면 앱 전체에 영향을 미칠 수 있기 때문이다.

# 해결책 2

이런 경우 액티비티에 적용 할 수 있다.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
        supportRequestWindowFeature(Window.FEATURE_ACTION_MODE_OVERLAY)
				super.onCreate(savedInstanceState)
}
```

onCreate 함수에 super 전에 위의 코드를 삽입하면 된다.

이럴 경우 앱전체에는 영향이 없고 해당 액티비티에만 적용이 된다.

이렇게 적용하면 액션바의 show hide를 해주는 코드도 필요가 없다.

전환시 문제없이 액션모드로 replace 된다.
