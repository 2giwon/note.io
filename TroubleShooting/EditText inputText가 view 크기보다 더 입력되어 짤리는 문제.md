
# 문제

커스텀 뷰 안에 EditText가 있는 경우 Edit

![image](https://github.com/2giwon/note.io/blob/master/TroubleShooting/resource/EditTextView.png)

위와 같이 EditText가 특정 뷰 안에 있고 입력이 길어지면 뷰보다 커져서입력 내용이 짤리게 된다.

안드로이드에서 이런 문제를 해결하기 위해서아래와 같이 EditText에 속성값을 넣어주면 된다.

```kotlin
android:maxLength="20"
android:singleLine="true"
```

그러나 이런 속성을 넣어도 폰트나 다른 나라 언어에 따라 입력시 짤리는 문제는 여전히 발생한다.

# 해결 방법 1

첫번째 해결 방법으로는 아무래도 각 언어와 폰트, 폰트 크기에 따라 길이가 달라지기 때문에

입력된 텍스트 뷰의 길이를 파악할 필요가 있다.

다행히도 안드로이드 paint에서는 이러한 문제를 해결해줄 api를 제공한다.

```kotlin
paint.measureText(text)
```

이 api는 입력된 text의 실제 뷰 width를 반환해준다.

또한 아래의 api도 있다.

```kotlin
val bounds = Rect()
paint.getTextBounds(text, 0, text.length, bounds)
bounds.width()
```

이 api도 위와 동일하다. 

그러나 주의해야 할 사항이 paint에 반드시 폰트 크기 값을 넣어줘야 정확한 길이값을 알 수 있다.

```kotlin
val paint = Paint().apply {
		textSize = editText.textSize
}
```

그럼 api도 있겠다. 

모두 해결된 것 같지만 길이를 안다고 해도 뷰 바깥까지 입력이 되는 text를 막을 방법이 없다.
