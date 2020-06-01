
# 문제

커스텀 뷰 안에 EditText가 있는 경우 Edit

![]()

위와 같이 EditText가 특정 뷰 안에 있고 입력이 길어지면 뷰보다 커져서입력 내용이 짤리게 된다.

안드로이드에서 이런 문제를 해결하기 위해서아래와 같이 EditText에 속성값을 넣어주면 된다.

```kotlin
android:maxLength="20"
android:singleLine="true"
```

그러나 이런 속성을 넣어도 폰트나 다른 나라 언어에 따라 입력시 짤리는 문제는 여전히 발생한다.
