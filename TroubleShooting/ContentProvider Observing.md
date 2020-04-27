## Intro
ContentProvider는 다른 앱 간에 통신을 위해서 사용할 수 있습니다.

두 개의 앱 사이에 공용 DB를 생성하고 DB안에 내용이 업데이트 되었을 때
플랫폼은 수신 앱에 알려주어 다른 처리를 할 수 있게 해줍니다.

![Figure.1](https://t1.daumcdn.net/cfile/tistory/22610D3B57E777F032)
![Figure.2](https://developer.android.com/guide/topics/providers/images/content-provider-tech-stack.png)

### Content Resolver

- 프로바이더 사이에서 중개자 역할
- query, insert, update, delete 작업이 가능

```
ContentResolver resolver = getContentResolver();
Cursor cursor = resolver, query(DroidTermsExampleContract.CONTENT_URI,
              null, null, null, null);
```
```
// Queries the user dictionary and returns results
cursor = contentResolver.query(
        UserDictionary.Words.CONTENT_URI,   // The content URI of the words table
        projection,                        // The columns to return for each row
        selectionClause,                   // Selection criteria
        selectionArgs.toTypedArray(),      // Selection criteria
        sortOrder                          // The sort order for the returned rows
)
```

사용해보시면 아시겠지만 다른 앱간의 통신이 양방향이 아닙니다.
송신앱과 수신앱이 있고 단방향으로 흘러갑니다.

그런데 수신 앱에서 변경 사실을 다른 송신 쪽 앱에서 알아야한다면?

## ContentObserver

콘텐츠 옵저버는 수신앱이 DB에 데이터를 변경할 경우 수신앱에서 알아차릴 수 있도록 제공하는 인터페이스 입니다.

아래의 코드처럼 옵저버를 등록해야 됩니다.
```java
context.getContentResolver().registerContentObserver(CONTENT_URI, true, mObserver);
```

mObserver는 아래의 클래스로 ContentObserver를 상속하여 만듭니다.

```java
private class SampleContentObserver extends ContentObserver {

   public SampleContentObserver () {
      super(new Handler());
   }

   public void onChange(final boolean selfChange) {
      updateFromProvider();
   }
}
```

수신앱에서 변경된 데이터를 db에 update하면 ContentProvider에서 등록된 옵저버에 변경된 사실을 알립니다.

옵저버는 변경된 사실을 받은 뒤 onChange 메소드를 호출 합니다.

onChange에서 변경되었을때의 처리를 하고
해제는 다음 코드를 확인해주세요.

```java
context.getContentResolver().unregisterContentObserver(mObserver);
```
