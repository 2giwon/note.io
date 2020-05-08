두 개의 변수를 동시에 초기화를 해야하는 경우가 있다.

나는 보통 소스 코드를 좀 더 간결하게 바꾸고 싶을때 였다.

같은 코드를 if문으로 나누어서 변수값만 다르게 들어가는 경우 (다들 이런경우 많을 것이다)

```
if (A) {
  functionA (VAR v)
} else {
  functionA (VAR a)
}
```

위의 예시는 다른 방법으로 할 수 있겠지만 그냥 이해를 돕기 위한 예시 일 뿐이다.

아래의 코드를 확인해보자.

```
  if (overlay != null) {
        backgroundAlpha = a.getFraction(
            R.styleable.Layout_background_alpha,
            1,
            1,
            0.5f
        ) * 255f
        withBorder = a.getBoolean(
            R.styleable.Overlay_with_border,
            true
        )
    } else {
        backgroundAlpha = 0.5f
        withBorder = true
    }
```


위의 코드는 overlay가 null 아닐 경우와 null경우에 대해서 각각의 값을 셋팅해주고 있다.

위의 코드를 처음엔 함수로 빼려 했다.

함수로 빼려니 동일한 동작을 2번하게 되어버려서 코드는 간결해질 지 몰라도 

성능상은 썩 좋지 않아 보였다.

다른 방법은?

지금으로써는 생각나지 않았다

함수로 빼서 코드를 간결하게 만들면서 동일한 동작을 2번씩 하지 않는 방법?

아래의 코드로 해결했다.

```
val (backgroundAlpha, withBorder) =
            initBackgroundAlphaNWithBorder(cropOverlayAttrs)

private fun initBackgroundAlphaNWithBorder(): Pair<Float, Boolean> {
        return if (cropOverlayAttrs != null) {
            typedArray.getFraction(
                R.styleable.CropOverlay_crop_background_alpha,
                1,
                1,
                0.8f
            ) * 255f to typedArray.getBoolean(
                R.styleable.CropOverlay_crop_with_border,
                true
            )
        
        } else {
            0.8f to true
        }
    }
```

위와 같이 하면 두 개의 변수를 동시에 초기화 하면서 함수로 뺄 수 있다


