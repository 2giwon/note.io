
# 문제

커스텀 뷰 안에 EditText가 있는 경우 Edit

![image](https://github.com/2giwon/note.io/blob/master/TroubleShooting/resource/EditTextView.png)

위와 같이 EditText가 특정 뷰 안에 있고 입력이 길어지면 뷰보다 커져서 입력 내용이 짤리게 된다.

안드로이드에서 이런 문제를 해결하기 위해서 아래와 같이 EditText에 속성값을 넣어주면 된다.

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

# 해결 방법 2

TextWatcher를 사용하여 입력되기 전 (`beforeTextChanged` 를 이용해 실제 뷰에 텍스트가 그려지기 전)에 길이를 파악하여 입력을 막아보자.

```kotlin
editText.addTextChangedListener(object : TextWatcher{
        override fun afterTextChanged(s: Editable?) {
						...
        }

        override fun beforeTextChanged(s: CharSequence?, start: Int, count: Int, after: Int) {
						val length = getMeasureTextLength(s.toString())
						if (length > width) {
		            editText.text = s.toString()
		        }
						
        }

        override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) {
						...
        }

    })
```

위의 코드가 문제가 있어보이나?

사실 위의 코드는 **매우 큰 문제가 있다!**

절대 위의 코드 처럼 해서는 안된다. 

왜냐하면 editText에 text 값을 넣어주면 다시 `TextWatcher`가 호출을 반복하기 때문에

`StackOverFlowException`이 발생할 수 있다.

# 해결 방법 2-1

안드로이드 editText API 중에는 inputFilter가 있다.

입력된 text가 뷰에 그려지기 전에 한번 걸러주는 api이다.

아래와 같이 filter를 구성하면 된다.

```kotlin
fun AppCompatEditText.blockMeasureTextLengthExceedFilter(): InputFilter {
    return InputFilter { _, _, _, dest, _, _ ->
        paint.textSize = textSize
        val measureText: Int = getMeasureTextWidth(dest.toString())

        if (measureText > width) {
            ""
        } else {
            null
        }
    }
}

fun AppCompatEditText.getMeasureTextWidth(text: String): Int {
    val bounds = Rect()
    paint.getTextBounds(text, 0, text.length, bounds)
    return bounds.width()
}
```

아직까지 문제는 없다

끝
