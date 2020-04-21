롤리팝 단말에서 WebVIew를 Inflate 할 때 아래와 같은 Exception 을 확인 할 수 있다.

```
java.lang.RuntimeException: Unable to start activity ComponentInfo{abc.abcd.ab/com.abcd.WebViewActivity}: android.view.InflateException: Binary XML file line #12: Error inflating class android.webkit.WebView
        at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2298)
        at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2360)
        at android.app.ActivityThread.access$800(ActivityThread.java:144)
        at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1278)
        at android.os.Handler.dispatchMessage(Handler.java:102)
        at android.os.Looper.loop(Looper.java:135)
        at android.app.ActivityThread.main(ActivityThread.java:5221)
        at java.lang.reflect.Method.invoke(Native Method)
        at java.lang.reflect.Method.invoke(Method.java:372)
        at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:899)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:694)
```

만약 AndroidX를 사용하고 있다면

버전을 확인 해볼 필요가 있다.

`1.1.0`은 webview widget에 대한 버그를 처리하지 않는다.

```
"androidx.appcompat:appcompat:1.0.2"
```
위의 버전으로 되돌리면 해결 할 수 있다.

이후에 구글 issue tracker에서 이슈를 reporting 하여 수정되었다.

https://issuetracker.google.com/issues/141132133


이슈 tracking 이후 이전 버전에서 동작하지 않고 최신버전인

```
androidx.appcompat:appcompat:1.2.0-alpha02
```

에서 해결된 모습을 볼 수 있다.

### 다른 해결책

커스텀한 webview를 생성하여 해결할 수 있다.

```
public class SbWebView extends WebView {
  public SbWebView(Context context) {
    super(getFixedContext(context));
  }
    
  public SbWebView(Context context, AttributeSet attrs) {
    super(getFixedContext(context), attrs);
  }
    
  public SbWebView(Context context, AttributeSet attrs, int defStyleAttr) {
    super(getFixedContext(context), attrs, defStyleAttr);
  }
    
  private static Context getFixedContext(Context context) {
    if (Build.VERSION.SDK_INT == 21 || Build.VERSION.SDK_INT == 22) // Android Lollipop 5.0 & 5.1
      return context.createConfigurationContext(new Configuration());
    return context;
  }
}
```
