[](https://developer.android.com/topic/performance/graphics/load-bitmap#load-bitmap)

# Loading Large Bitmaps Efficiently

Note: 이미지 로드에 모범 사례를 따르는 여러 라이브러리가 있습니다.
앱에서 이 라이브러리를 사용하여 이미지를 최적화 된 방식으로 로드할 수 있습니다.
이런 라이브러 사용을 권장합니다.

1. 이미지는 크기가 모양이 다름
2. 일반적인 경우 UI 보다 사이즈가 큼
3. 카메라로 찍은 사진은 밀도가 높음
4. 제한된 메모리 상에서는 낮은 해상도로 로드 하는게 이상적
5. 샘플링 버전을 메모리에 로드하여 최적의 비트맵 디코딩 과정을 적용하자

## Read Bitmap Dimensions and Type

- BitmapFactory 는 다양한 디코딩 메소드를 제공
- 디코딩 메소드는 OOM (OutOfMemory) Exception이 발생할 수 있음
- `BitmapFactory.Options` 를 통해 디코딩 옵션을 지정할 수 있음
- `inJustDecodeBounds` 속성은 메모리 할당을 피하여 비트맵 객체에 null을 반환
- outWidth, outHeight, outMimeType을 설정하기 때문에 비트맵을 메모리에 할당하기 전에 크기와 유형을 읽을 수 있음

```kotlin
val options = BitmapFactory.Options().apply {
    inJustDecodeBounds = true
}
BitmapFactory.decodeResource(resources, R.id.myimage, options)
val imageHeight: Int = options.outHeight
val imageWidth: Int = options.outWidth
val imageType: String = options.outMimeType
```

OOM을 피하기 위해 예측 크기를 통해 비트맵 크기를 확인!

## Load a Scaled Down Version into Memory

- 예측 크기를 통해 샘플 크기로 로드할 지 전체를 로드 할 지 결정
- 썸네일의 경우 원본 해상도로 로드할 필요가 없음
- 디코더가 이미지를 서브 샘플링으로 더 작게 로드하도록 `BitmapFactory.Options`로 `inSampleSize`를 true로 설정
- inSampleSize는 예를 들어 4로 설정하면 비트맵 바이너리 맵에서 4만큼 건너 뛰어서 샘플링한 데이터를 가지고 비트맵을 형성하여 메모리를 줄일 수 있음
- 2048x1536의 이미지는 약 512x384의 비트 맵
- 전체 이미지에 12MB가 아닌 0.75MB가 메모리로 사용 (ARGB_8888의 비트 맵 구성 가정)
- 계산 코드

```kotlin
fun calculateInSampleSize(options: BitmapFactory.Options, reqWidth: Int, reqHeight: Int): Int {
    // Raw height and width of image
    val (height: Int, width: Int) = options.run { outHeight to outWidth }
    var inSampleSize = 1

    if (height > reqHeight || width > reqWidth) {

        val halfHeight: Int = height / 2
        val halfWidth: Int = width / 2

        // Calculate the largest inSampleSize value that is a power of 2 and keeps both
        // height and width larger than the requested height and width.
        while (halfHeight / inSampleSize >= reqHeight && halfWidth / inSampleSize >= reqWidth) {
            inSampleSize *= 2
        }
    }

    return inSampleSize
}
```

위의 방법을 사용하려면 `inJustDecodeBounds` 를 true로 설정

디코딩 후 `inSampleSize` 와 `inJustDecodeBounds`를 다시 false로 설정하여 디코드

```kotlin
fun decodeSampledBitmapFromResource(
        res: Resources,
        resId: Int,
        reqWidth: Int,
        reqHeight: Int
): Bitmap {
    // First decode with inJustDecodeBounds=true to check dimensions
    return BitmapFactory.Options().run {
        inJustDecodeBounds = true
        BitmapFactory.decodeResource(res, resId, this)

        // Calculate inSampleSize
        inSampleSize = calculateInSampleSize(this, reqWidth, reqHeight)

        // Decode bitmap with inSampleSize set
        inJustDecodeBounds = false

        BitmapFactory.decodeResource(res, resId, this)
    }
}
```

이 방법을 사용하면 다음 예제 코드와 같이 100x100 픽셀 축소판을 표시하는 ImageView에 임의로 큰 크기의 비트 맵을 쉽게 로드 할 수 있습니다.

```kotlin
imageView.setImageBitmap(
        decodeSampledBitmapFromResource(resources, R.id.myimage, 100, 100)
)
```
