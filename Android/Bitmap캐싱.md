```
참고 : 대부분의 경우 Glide 라이브러리를 사용하여 앱에서 비트맵을 가져오고 디코딩하고 표시하는것을 추천
Glide는 이러한 작업을 비롯하여 Android에서 비트맵과 기타 이미지를 사용하는 다른 관련 작업을 처리할 때 대부분의 복잡성을 추상화합니다.
```

# 메모리 캐시 사용

- 메모리 캐시는 중요한 애플리케이션 메모리 대신 비트맵에 빠르게 액세스 가능
- LruCache 클래스는 비트맵을 캐싱
- LruCache는 최근에 참조된 객체를 강한참조함
- LinkedHashMap에 유지하는 작업도함
- 캐시에 지정된 크기를 초과하기 전에 가장 오래전에 사용된 항목을 제거

이전에는 SoftReference또는 WeakReference 비트맵 캐시가 메모리 캐시 구현으로 많이 사용되었음. **지금은 권장하지 않음** 
안드로이드 2.3 부터 가비지 컬렉터가 소프트/약한 참조를 적극적으로 수집하여 이러한 참조 효과를 떨어뜨림
또한 3.0 이전에는 비트맵의 백업데이터가 예측할 수 있는 방식으로 해제되지 않은 네이티브 메모리에 저장되어 애플리케이션이 잠시 메모리 제한을 초과하여 비정상 종료될 수 있는 가능성이 잠재되어 있음.

## LruCache의 적합한 크기 정하기

- Activity 및 Application의 나머지 부분이 얼마나 많은 메모리를 사용하는가
- 한 번에  몇 개의 이미지가 화면에 표시되나?
- 화면에 표시할 준비가 되어야할 이미지 수는?
- 기기의 화면의 밀도는?
- 초고밀도 화면(xhdpi)기기는 메모리에 동일한 이미지를 저장하려면 더 많은 캐시가 필요함
- 비트맵의 크기와 구성은 어떻게 되나? 메모리 사용량은?
- 이미지는 얼마나 자주 액세스 되나?
- 품질과 수량의 균형을 맞출 수 있나? 때로는 더 낮은 품질의 비트맵을 다수 저장하여 잠재적으로 백그라운드 작업에 더 높은 품질 버전을 로드할 수 있도록 하는것이 더 유용함.

캐시가 너무 작으면 이점이 없고

너무 크면 오버헤드가 발생함(OOM)

## LruCache 설정 방법

```kotlin
private lateinit var memoryCache: LruCache<String, Bitmap>

    override fun onCreate(savedInstanceState: Bundle?) {
        ...
        // Get max available VM memory, exceeding this amount will throw an
        // OutOfMemory exception. Stored in kilobytes as LruCache takes an
        // int in its constructor.
        val maxMemory = (Runtime.getRuntime().maxMemory() / 1024).toInt()

        // Use 1/8th of the available memory for this memory cache.
        val cacheSize = maxMemory / 8

        memoryCache = object : LruCache<String, Bitmap>(cacheSize) {

            override fun sizeOf(key: String, bitmap: Bitmap): Int {
                // The cache size will be measured in kilobytes rather than
                // number of items.
                return bitmap.byteCount / 1024
            }
        }
        ...
    }
```
```
참고: 이 예에서는 애플리케이션 메모리의 1/8이 캐시로 할당됩니다. 
일반/hdpi 기기의 경우 최소 4MB 정도입니다. 
해상도가 800x480인 기기에서 이미지로 채워진 전체 화면 GridView는 약 1.5MB(800*480*4바이트)를 사용하므로 최소 약 2.5페이지의 이미지를 메모리에 캐시합니다.
```

- 비트맵을 ImageView에 로드할 때 먼저 LruCache가 확인됨.
- 발견되면 ImageView에 즉시 사용
- 없으면 백그라운드에서 생성

```kotlin
fun loadBitmap(resId: Int, imageView: ImageView) {
    val imageKey: String = resId.toString()

    val bitmap: Bitmap? = getBitmapFromMemCache(imageKey)?.also {
        mImageView.setImageBitmap(it)
    } ?: run {
        mImageView.setImageResource(R.drawable.image_placeholder)
        val task = BitmapWorkerTask()
        task.execute(resId)
        null
    }
}
```

```kotlin
private inner class BitmapWorkerTask : AsyncTask<Int, Unit, Bitmap>() {
    ...
    // Decode image in background.
    override fun doInBackground(vararg params: Int?): Bitmap? {
        return params[0]?.let { imageId ->
            decodeSampledBitmapFromResource(resources, imageId, 100, 100)?.also { bitmap ->
                addBitmapToMemoryCache(imageId.toString(), bitmap)
            }
        }
    }
    ...
}
```

# 디스크 캐시 사용

메모리 캐시에서 사용하는 비트맵은 애플리케이션이 중단되면 없어 질 수 있으며

많은 용량의 이미지를 캐시할 수 없습니다.

또한 계속해서 캐시를 갱신할 경우 다시 이미지를 생성할 때 시간이 소요됩니다.

디스크 캐시는 비트맵을 유지할 수 있고 더이상 사용하지 않을 때 저장하여 로드 시간을 줄일 수 있습니다.

메모리 캐시보다는 속도는 느립니다.

항상 백그라운드에서 동작해야됩니다.

이미지가 이미지 갤러리 애플리케이션 같은 곳에서 자주 액세스된다면 캐시된 이미지를 저장하기에 ContentProvider가 더 적합한 장소가 될 수 있습니다

## DiskLruCache 예제

```kotlin
private const val DISK_CACHE_SIZE = 1024 * 1024 * 10 // 10MB
private const val DISK_CACHE_SUBDIR = "thumbnails"
...
private var diskLruCache: DiskLruCache? = null
private val diskCacheLock = ReentrantLock()
private val diskCacheLockCondition: Condition = diskCacheLock.newCondition()
private var diskCacheStarting = true

override fun onCreate(savedInstanceState: Bundle?) {
    ...
    // Initialize memory cache
    ...
    // Initialize disk cache on background thread
    val cacheDir = getDiskCacheDir(this, DISK_CACHE_SUBDIR)
    InitDiskCacheTask().execute(cacheDir)
    ...
}

internal inner class InitDiskCacheTask : AsyncTask<File, Void, Void>() {
    override fun doInBackground(vararg params: File): Void? {
        diskCacheLock.withLock {
            val cacheDir = params[0]
            diskLruCache = DiskLruCache.open(cacheDir, DISK_CACHE_SIZE)
            diskCacheStarting = false // Finished initialization
            diskCacheLockCondition.signalAll() // Wake any waiting threads
        }
        return null
    }
}

internal inner class  BitmapWorkerTask : AsyncTask<Int, Unit, Bitmap>() {
    ...

    // Decode image in background.
    override fun doInBackground(vararg params: Int?): Bitmap? {
        val imageKey = params[0].toString()

        // Check disk cache in background thread
        return getBitmapFromDiskCache(imageKey) ?:
                // Not found in disk cache
                decodeSampledBitmapFromResource(resources, params[0], 100, 100)
                        ?.also {
                            // Add final bitmap to caches
                            addBitmapToCache(imageKey, it)
                        }
    }
}

fun addBitmapToCache(key: String, bitmap: Bitmap) {
    // Add to memory cache as before
    if (getBitmapFromMemCache(key) == null) {
        memoryCache.put(key, bitmap)
    }

    // Also add to disk cache
    synchronized(diskCacheLock) {
        diskLruCache?.apply {
            if (!containsKey(key)) {
                put(key, bitmap)
            }
        }
    }
}

fun getBitmapFromDiskCache(key: String): Bitmap? =
        diskCacheLock.withLock {
            // Wait while disk cache is started from background thread
            while (diskCacheStarting) {
                try {
                    diskCacheLockCondition.await()
                } catch (e: InterruptedException) {
                }

            }
            return diskLruCache?.get(key)
        }

// Creates a unique subdirectory of the designated app cache directory. Tries to use external
// but if not mounted, falls back on internal storage.
fun getDiskCacheDir(context: Context, uniqueName: String): File {
    // Check if media is mounted or storage is built-in, if so, try and use external cache dir
    // otherwise use internal cache dir
    val cachePath =
            if (Environment.MEDIA_MOUNTED == Environment.getExternalStorageState()
                    || !isExternalStorageRemovable()) {
                context.externalCacheDir.path
            } else {
                context.cacheDir.path
            }

    return File(cachePath + File.separator + uniqueName)
}
```
```
참고: 디스크 캐시를 초기화하는 것도 디스크 작업이 필요하므로 기본 스레드에서 실행하면 안 됩니다. 
하지만 이는 초기화 전에 캐시에 액세스할 기회가 있음을 의미합니다. 
이 문제를 해결하기 위해 위의 구현에서 잠금 객체는 캐시가 초기화될 때까지 앱이 디스크 캐시에서 읽지 않도록 합니다.
```
