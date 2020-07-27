# 개요

안드로이드 10 을 지원하면서 이전에 startService 로 동작했던 서비스들이 동작하지 않는 문제가 발생했다.

원인은 service를 background로 동작해서는 안되는 것.

이 문제는 oreo에서 부터 적용되었던 사항인데 그동안에는 안드로이드에서 허용이 되었었다. 안드로이드 10에서 부터는 허용되지 않은것!

10부터는 service는 foregroundService로 동작해야 된다.

# 수정 사항

- 일단 sdk 버전이 29인지 확인 한다.

    ```kotlin
    compileSdkVersion 29
    ```

- Android Manifest에서 아래처럼 permission을 등록한다.

    ```kotlin
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
    ```

- `<service>` 를 사용하는 태크에 foregroundServiceType으로 mediaProjection이 적용되어 잇는지 확인 한다.

    ```kotlin
    android:foregroundServiceType="mediaProjection"
    ```

- 코드 상에서 startService 부분을 변경한다.

    ```kotlin
    if (Build.VERSION.SDK_INT > Build.VERSION_CODES.P) {
    	Objects.requireNonNull(mContext).startForegroundService(intent);
    } else {
    	Objects.requireNonNull(mContext).startService(intent);
    }
    ```

그리고 가장 중요한 Notification을 설정해야한다.

다 위처럼 구성했는데도 동작하지 않아 애를 먹었었다.

문제는 Notification을 설정해주어야 한다.

이 부분은 서비스로 동작할 때 사용자에게 알림으로 알려주는 것이다.

# Notification 설정

Service 클래스 내로 이동하자.

onCreate내에 아래와 같이 Notification Channel을 생성해야한다.

```kotlin
@Override
public void onCreate() {
	super.onCreate();
	
	...
		
	...
	if (android.os.Build.VERSION.SDK_INT > android.os.Build.VERSION_CODES.P) {
		String channelId = createNotificationChannel();
		mNotification = createNotificationBuilder(channelId);
	}
}
```

```kotlin
@RequiresApi(api = Build.VERSION_CODES.O)
private String createNotificationChannel() {
	NotificationChannel channel = new NotificationChannel(ScreenCaptureServcie.CHANNEL_ID, "Capture_service", NotificationManager.IMPORTANCE_NONE);

	channel.setLightColor(Color.BLUE);
	channel.setLockscreenVisibility(Notification.VISIBILITY_PRIVATE);

	mNotificationManager = (NotificationManager)getSystemService(NOTIFICATION_SERVICE);
	if (mNotificationManager != null) {
		mNotificationManager.createNotificationChannel(channel);
		return ScreenCaptureServcie.CHANNEL_ID;
	}

	return "";
}
```

생성된 채널 ID값으로 Notification.Builder를 통해 Notification을 생성한다.

```kotlin
private Notification createNotificationBuilder(String channelID) {
	if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
		return new Notification.Builder(this, channelID)
				.setContentTitle("Foreground Service")
				.setContentText("ScreenCaptureService")
				.build();
	} else {
		return new Notification.Builder(this).build();
	}
}
```

# StartForegound

그 다음 startForeground 를 선언해줘야 한다. 이 부분은 startForegroundService를 선언 해주고

5초 안에 이루어 져야 한다. 

5초안에 startForeground를 선언하지 않으면 플랫폼에서는 exception을 발생시킨다.

Service 클래스의 onStartCommand 내에 아래와 같이 startForeground를 선언한다.

```kotlin
if (android.os.Build.VERSION.SDK_INT > android.os.Build.VERSION_CODES.P) {
	startForeground(NOTIFICATION_ID, mNotification);
}
```

startForeground를 startForegroundService와 혼동하는 경우가 있는데

둘 다 선언해야한다.
