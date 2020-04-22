9월 3일 Android 10은 공식 출시 되었습니다.

논란의 여지가 있는 변경 사항 Scoped Storage가 있습니다.

간단히 말해서, Scoped Storage는 앱이 장치의 파일 시스템에 무제한으로 액세스하는 것을 방지합니다. 

이전에는 앱이 표준 파일 API를 사용하여 기기의 모든 파일에 액세스 할 수 있었으며 사용자에게만 저장소 권한을 부여해야했습니다.

## Android 10의 모든 앱에 Scoped Storage가 필요합니까?

targetSdkVersion이 29로 설정된 경우에만 적용되므로 레거시 앱이 계속 작동합니다. 

또한 API 29를 대상으로하는 경우에도`AndroidManifest.xml` 내부의 애플리케이션 태그에서

```
android:requestLegacyExternalStorage = "true"
```

를 설정하여 레거시 스토리지를 계속 사용할 수 있습니다.

그러나 Android 11에서 가능한 한 Scoped Storage를 구현해야합니다.
Google은 targetSdkVersion에 관계없이 모든 앱에 필요하다고 말합니다.

## 간단한 파일에 액세스 하는 방법?

장치의 파일에 액세스하기 위해 SAF (Storage Access Framework)를 사용할 수 있습니다.
ACTION_OPEN_DOCUMENT를 사용하면 필요한 문서를 선택 할 수있는 대화 상자가 사용자에게 표시됩니다.

ACTION_OPEN_DOCUMENT_TREE에는 파일을 저장하기 위해 사용자에게 디렉토리를 선택하고 ACTION_CREATE_DOCUMENT를 요청하라는 메시지도 있습니다.

## 사진 응용 App 에서 Scoped Storage 사용

이제 출시 예정인 앱인 Date To Photo (Date To Photo)를 예로 들겠습니다. 
최근에 Scoped Storage와 호환되도록 업데이트 한 이후 예를 들었습니다.
Date To Photo는 사진을 촬영 한 날짜와 함께 사진에 워터 마크를 추가 할 수 있는 앱입니다.

사용자가 앱을 열면 아직 워터 마크되지 않은 사진이 있는 격자가 나타나고 
하나 이상의 사진을 선택하거나 모든 사진에 워터 마크를 추가 할 수 있습니다.

### 사진 갤러리 앱에서 표시

주의해야할 사항은 파일을 사용하지 않고 (MediaStore.Images.Media.DATA 열은 더이상 사용되지 않음) 대신 Uri 를 사용합니다.

`MediaStore.Images.Media._ID`를 사용하여 사진의 id를 얻고 
`ContentUris.withAppendedId`를 사용하여 Uri를 만들 수 있습니다.

- MediaStore.Images.Media.BUCKET_DISPLAY_NAME : 사진이 있는 폴더를 알 수 있음
- MediaStore.Images.Media.DISPLAY_NAME : 파일의 원래 이름을 알 수 있음

### 사진 처리

사진을 처리하기 위해 비트맵을 가져오려면 
Uri 에서 InputStream을 가져와서 BitmapFactory.decodeResource에 전달하거나 
API 28+ 에서 새로운 `ImageDecoder` API를 사용할 수 있습니다,

```
val bitmap = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
      ImageDecoder.decodeBitmap(ImageDecoder.createSource(contentResolver, uri))
    } else {
      contentResolver.openInputStream(uri)?.use { inputStream ->
        BitmapFactory.decodeStream(inputStream)
      }
    }
```

### 갤러리에 사진 저장

Scoped Storage 때문에, 이미지를 원하는 폴더에 저장할 수 없습니다, 그리고 MediaStore도 업데이트 할 수 없습니다.

대신 Android Q는 이미지의 경로를 지정할 수 있는 새로운 필드
(`MediaStore.Images.Media.RELATIVE_PATH`)를 도입합니다 (예 : "Pictures / Screenshots /").

위에 표시된대로 이미지 속성 및 상대 경로를 설정하는 것 외에도 새 필드 
`MediaStore.Images.Media.IS_PENDING` 을 설정하고 이를 MediaStore에 삽입해야 됩니다.

```
  val values = ContentValues().apply {
        put(MediaStore.Images.Media.DISPLAY_NAME, name)
        put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg")
        put(MediaStore.Images.Media.RELATIVE_PATH, "Pictures/$bucketName/")
        put(MediaStore.Images.Media.IS_PENDING, 1)
    }
    
    val collection = MediaStore.Images.Media.getContentUri(MediaStore.VOLUME_EXTERNAL_PRIMARY)
    val imageUri = context.contentResolver.insert(collection, values)
```

위에서 확인 되는 것은 이미지 저장을 위해 MediaStore.Images.Media.EXTERNAL_CONTENT_URI

를 사용하지 않는 다는 것입니다.

대신, Android Q를 사용하면 외부 sdcard와 같은 외부 저장 장치에 미디어 파일을 쉽게 저장할 수 있습니다.

여기서는 MediaStore.VOLUME_EXTERNAL_PRIMARY를 지정하여 이미지를 장치의 기본 저장소에 저장하지만 MediaStore.getExternalVolumeNames  를 사용하여 모든 저장 장치 목록을 가져올 수 있습니다.

그런 다음 반환 된 이미지 Uri를 사용하여 이미지를 저장할 수 있습니다.

이를 위해

```
  context.contentResolver.openOutputStream
```

을 사용하여 해당 스트림에 저장하거나
```
  context.contentResolver.openFileDescriptor
```
 를 사용 하여 쓰기 모드에서 FileDescriptor를 열 수도 있습니다.

마지막으로 이미지가 저장되면 MediaStore.Images.Media.IS_PENDING 속성을 0으로 설정하여 이미지 삽입이 완료되었음을 나타냅니다.


```
  context.contentResolver.openOutputStream(imageUri).use { out ->
        bmp.compress(Bitmap.CompressFormat.JPEG, 90, out)
    }
    
    values.clear()
    values.put(MediaStore.Images.Media.IS_PENDING, 0)
    context.contentResolver.update(imageUri, values, null, null)
```

### 원본 사진 덮어쓰기

원본 이미지를 덮어 쓰려면 Uri에 쓰거나 File Descriptor를 가져와야 됩니다.

**다른 App에서 추가한 이미지에서 update를 호출하거나 쓰기 모드에서 File Descriptor를 삭제 또는 열려고 하면 
RecoverableSecurityException을 발생시킵니다.
try/catch로 포착을 하고 사용자에게 액세스 권한을 요청하면 됩니다.**

```
try {
    context.openOutputStream(imageUri).use { out ->
        bmp.compress(Bitmap.CompressFormat.JPEG, 90, out)
    }                                       
} catch (e: RecoverableSecurityException) {
    handleRecoverableSecurityException(e)
}
```

```
AlertDialog.Builder(requireActivity())
		.setMessage(e.userMessage)
	  .setPositiveButton(e.userAction.title) { dialog, which -> 
		    try {
		        e.userAction.actionIntent.send()
				} catch (ignored: PendingIntent.CanceledException) {}
		}
    .setNegativeButton(android.R.string.cancel, null)
		.show()
```

이 방법은 하나의 이미지 삭제에는 괜찮지만, 

여러 항목을 조작해야 되는 상황이거나 Background에서 이미지를 처리해야하는 경우는 적합하지 않습니다.

이런 경우

```
val intent = Intent(Intent.ACTION_OPEN_DOCUMENT_TREE)
intent.addFlags(Intent.FLAG_GRANT_WRITE_URI_PERMISSION)
intent.addFlags(Intent.FLAG_GRANT_PERSISTABLE_URI_PERMISSION)
// You can specify initial folder using intent.putExtra(DocumentsContract.EXTRA_INITIAL_URI
startActivityForResult(intent, REQUEST_CODE)
```

사용자에게 액세스 권한을 부여한 후 지속 가능 한 권한을 가져옵니다.

```
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    if (requestCode == REQUEST_CODE && resultCode == Activity.RESULT_OK) {
        val uri = data?.data
        uri?.let { contentResolver.takePersistableUriPermission(it, Intent.FLAG_GRANT_WRITE_URI_PERMISSION) }
    }
}
```

사진을 덮어 쓰려면 Uri에 덮어 쓸 수 없고 SAF Uri로 변환 해야 합니다.
```
val documentUri = MediaStore.getDocumentUri(context, imageUri)
context.contentResolver.openOutputStream(documentUri).use { out ->
    bmp.compress(Bitmap.CompressFormat.JPEG, 90, out)
}
```


