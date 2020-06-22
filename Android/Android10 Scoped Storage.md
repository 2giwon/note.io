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
```kotlin
fun addObserve() {
    viewModel.directoryUri.observe(this, Observer { event ->
        event.getContentIfNotHandled().let {
            val documentFile: DocumentFile =
                DocumentFile.fromTreeUri(this@FileBrowserActivity, it) ?: return@Observer
            viewModel.loadDirectory(documentFile)
        }
    })
}
```

DocumentFile에는 파일의 각종정보가 들어있습니다. id, 이름, 타입, 용량 등등...

그럼 다 해결한 것 같습니다. 파일브라우저 만드는 건 어렵지 않네요!

실행해보시면 아직 멀었다는 것을 알 수 있습니다.

**속도가 매우매우 느립니다!**

root 경로를 탐색했을때 걸리는 시간은 제 폰 기준으로 19~20초 정도 걸렸습니다.

이대로 사용한다면 사용성에 문제가 큽니다.

문제는 DocumentFile에서 listFiles() 를 하면 파일 리스트를 생성하는데 시간이 굉장히 오래 걸리는 것 입니다.

### 해결 방법

1. RxJava를 통해서 파일 하나를 읽어 올때마다 화면에 보여주어서 사용자에게 

처리되고 있다는 화면을 보여준다.

```kotlin
fun loadDirectory(documentFile: DocumentFile) {
    Observable.create<DocumentItem> { emit ->
        _documentList.clear()
        documentFile.listFiles().forEach { emit.onNext(DocumentItem(it)) }
        emit.onComplete()
    }
        .doOnSubscribe { _loadingBar.postValue(true) }
        .doOnComplete { _loadingBar.postValue(false) }
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribeBy(
            onNext = {
                _documentList.add(it)
                _documents.value = _documentList
            },

            onComplete = {
                _documents.value = _documentList
                    .asSequence()
                    .filter { !(it.name?.startsWith(".") ?: true) }
                    .sortedWith(compareBy({ !it.isDirectory }, { it.name?.toLowerCase() }))
                    .toList()
            }
        )
        .addTo(compositeDisposable)
}
```

파일 한 개씩 가져와서 뿌려주고 로딩이 끝나면 .으로 시작하는 폴더를 제거하고 오름차순으로 정렬하도록 했습니다.

결과는 만족스러운 결과를 보여주지만 그래도 아직 실제 속도는 느리기 때문에 좋은 해결방법은 아닙니다.

2. listFiles를 확인 해보면 하나의 항목(이름이라던가 용량등)을 요청할 때마다 ContentResolver를 통해서 요청을 하게 되어있습니다. 

이부분이 처음에는 굉장히 이상했습니다.

그렇다면 직접 ContentResolver를 통해서 필요한 항목을 요청하면 되는 것이 아닌가?

## 개선된 파일 브라우저

이제 DocumentFile 따위는 필요 없습니다.

직접 요청합시다.

```kotlin
private fun loadDirectoryFromContentResolver(documentUri: Uri): List<DocumentItem> {

    val treeDocumentUri: Uri = getTreeDocumentUri(documentUri)

    val contentResolver: ContentResolver = applicationContext.contentResolver
    val childrenUri = DocumentsContract.buildChildDocumentsUriUsingTree(
        treeDocumentUri,
        DocumentsContract.getDocumentId(treeDocumentUri)
    )

    val result = mutableListOf<DocumentItem>()
    var cursor: Cursor? = null

    try {
        cursor = createCursor(contentResolver, childrenUri)

        cursor?.let { c ->
            while (c.moveToNext()) {
                val documentId = c.getDocumentID()
                val name = c.getDocumentName()
                val type = c.getDocumentType()
                val isDirectory: Boolean = type == DocumentsContract.Document.MIME_TYPE_DIR
                val lastModified = c.getLastModified().toLastModifiedTime()
                val uri = getDocumentUri(treeDocumentUri, documentId)
                val size = c.getDocumentSize().toFileSizeUnit()

                result.addDocumentItem(name, type, isDirectory, uri, size, lastModified)
            }

        }
    } catch (e: Exception) {
        throw e
    } finally {
        closeQuietly(cursor)
    }

    return result

}
```

```kotlin
private fun createCursor(contentResolver: ContentResolver, childrenUri: Uri): Cursor? {
    return contentResolver.query(
        childrenUri,
        arrayOf(
            DocumentsContract.Document.COLUMN_DOCUMENT_ID,
            DocumentsContract.Document.COLUMN_DISPLAY_NAME,
            DocumentsContract.Document.COLUMN_MIME_TYPE,
            DocumentsContract.Document.COLUMN_LAST_MODIFIED,
            DocumentsContract.Document.COLUMN_SIZE
        ), null, null, null
    )
}
```

ContentResolver를 통해서 필요한 항목을 컬럼으로 요청하여 Cursor를 받고

Cursor를 통해서 파일들을 가져오는 방식으로 처리합니다.

결과는?

20 초씩 걸리는 것이 1초로 줄어든 것을 확인 할 수 있습니다!!

자세한 사항은 아래 코드를 확인바랍니다.
[2giwon/ScopedStorageExample](https://github.com/2giwon/ScopedStorageExample)
