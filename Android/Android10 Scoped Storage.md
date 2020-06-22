# Android Scoped Storage

[Working with Scoped Storage](https://www.notion.so/midfgr/Working-with-Scoped-Storage-9edb8572d8444e28bbd2aa66d037f2bb)

## Android10 Scoped Storage 공용폴더

안드로이드10 스코프드 스토리지라도 공용폴더는 사용할 수 있습니다.

- 앱데이터 폴더
- 미디어 파일(MediaStore)
- Download 폴더 등

또한 Storage Access FrameWork(SAF)를 통해서 이러한 공용폴더의 접근을 할 수 있고

root 폴더에도 접근 할 수 있습니다.

## Android Storage의 데이터 접근 방식 변경

앞으로는 모두 SAF 를 통해서 안전하게 접근하도록 하는 것이 구글의 지향하는 방향으로 보입니다.

또한 직접적인 경로(Absolute path)를 통해서 접근하지 않고 Uri를 통해서만 데이터를 가져올 수 있도록 하고 있습니다.

## Android Scoped Storage를 이용한 파일 브라우저 만들기

안드로이드10에서 사용되는 Scoped Storage에 관련된 내용은 위의 링크를 확인하고

여기서는 간단한 샘플을 통해 Scoped Storage가 어떤 식으로 대응하면 되는지 알아보자.

### 안드로이드 10 이전 버전

```kotlin
android:requestLegacyExternalStorage="true"
```

위의 속성을 메니페스트에 선언합니다.

그리고 이전 방식으로 접근하여 

```java
File exSD = Environment.getExternalStorageDirectory();
```

경로를 획득하는 등의 방식으로 처리할 수 있습니다.

그럼 왜 이런 방식을 써야하는가?

안드로이드는 하위 호환성이 무엇보다 중요합니다.

그렇기 때문에 이전버전에서 사용하거나 

이전에 배포되었던 버전도 동일하게 하위 호환성이 중요하기 때문에 

예전에 사용하거나 레거시 코드로 존재했던 방식을 그대로 사용하는 것이 정신건강에 좋습니다.

### Android 10에서의 대응

```java
...
private fun openDirectory() {
    val intent = Intent(Intent.ACTION_OPEN_DOCUMENT_TREE).apply {
        flags = Intent.FLAG_GRANT_READ_URI_PERMISSION or
            Intent.FLAG_GRANT_PERSISTABLE_URI_PERMISSION
    }
    startActivityForResult(intent, OPEN_DIRECTORY_REQUEST_CODE)
}
...
```

위의 방식으로 파일 리스트를 SAF로 요청할 수 있습니다.

받아온 데이터를 어떻게 처리해야 하지?

간단한 방식으로 DocumentFile을 사용할 수 있습니다.
