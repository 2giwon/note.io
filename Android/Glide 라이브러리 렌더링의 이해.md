안드로이드 이미지 라이브러리 **[Glide](https://bumptech.github.io/glide/)**가 어떻게 이미지를 표시하게 되는지 

그 과정을 간략히 설명한다.

# 로딩 상태에 대응하는 Target

Glide에서 이미지를 로딩을 시작하면 [**Target**](https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/request/target/Target.java)을 통해 진행 상황을 알려온다.

```java
public interface Target<R> extends LifecycleListener {
  void onLoadStarted(@Nullable Drawable placeholder);
  void onLoadFailed(@Nullable Drawable errorDrawable);
  void onResourceReady(@NonNull R resource, @Nullable Transition<? super R> transition);
  void onLoadCleared(@Nullable Drawable placeholder);
  void getSize(@NonNull SizeReadyCallback cb);
  void removeCallback(@NonNull SizeReadyCallback cb);
  void setRequest(@Nullable Request request);
  Request getRequest();
}
```

이미지를 가져오기 시작할 때 `onLoadStarted`가 호출되고 

실패시 `onLoadFailed`, 

성공시 `onResourceReady`가 호출된다. 

`LifecycleListener`을 상속받았다는 것을 통해 안드로이드 라이프사이클에 대응하는 것도 `Target`의 책임이라는 것을 알 수 있다.

로딩 후 ImageView를 갱신하는 관련 Target이 몇가지 있는데 [**ImageViewTarget**](https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/request/target/ImageViewTarget.java)을 상속받는다.

리소스가 준비되면 `onResourceReady` 메서드가 호출된다.

```java
@Override
public void onResourceReady(@NonNull Z resource, @Nullable Transition<? super Z> transition) {
  if (transition == null || !transition.transition(resource, this)) {
    setResourceInternal(resource);
  } else {
    maybeUpdateAnimatable(resource);
  }
}
```

`setResourceInternal`를 따라가보자.

```java
private void setResourceInternal(@Nullable Z resource) {
  // Order matters here. Set the resource first to make sure that the Drawable has a valid and
  // non-null Callback before starting it.
  setResource(resource);
  maybeUpdateAnimatable(resource);
}
```

`setResource`가 호출된다.

`ImageViewTarget`의 자식 [BitmapImageViewTarget](https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/request/target/BitmapImageViewTarget.java)의 구현을 보자.

```java
@Override
protected void setResource(Bitmap resource) {
  view.setImageBitmap(resource);
}
```

이미지 뷰의 `setImageBitmap`으로 로딩된 비트맵을 적용했다.

```java
private void maybeUpdateAnimatable(@Nullable Z resource) {
  if (resource instanceof Animatable) {
    animatable = (Animatable) resource;
    animatable.start();
  } else {
    animatable = null;
  }
}
```

[**Animatable**](https://developer.android.com/reference/android/graphics/drawable/Animatable)인 경우 `Animatable#start`를 호출한다.

`Animatable`은 안드로이드 표준 객체인데 에니메이션을 정지하고 시작할 수 있는 기능을 제공하는 인터페이스다.

# 애니메이션 시작 GifDrawable#start

```java
public class GifDrawable extends Drawable
    implements GifFrameLoader.FrameCallback, Animatable, Animatable2Compat {
}
```

[**GifDrawable**](https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/load/resource/gif/GifDrawable.java)이 `Animatable`이기 때문에 구현된 `start` 메서드가 호출된다.

```java
@Override
public void start() {
  isStarted = true;
  resetLoopCount();
  if (isVisible) {
    startRunning();
  }
}
```

시작 상태로 체크하고, 루프 카운트를 리셋하고 visible한 상태일 때만 `startRunning`을 호출한다.

```java
private void startRunning() {
  Preconditions.checkArgument(
      !isRecycled,
      "You cannot start a recycled Drawable. Ensure that"
          + "you clear any references to the Drawable when clearing the corresponding request.");
  // If we have only a single frame, we don't want to decode it endlessly.
  if (state.frameLoader.getFrameCount() == 1) {
    invalidateSelf();
  } else if (!isRunning) {
    isRunning = true;
    state.frameLoader.subscribe(this);
    invalidateSelf();
  }
}
```

`Precondition.checkArgument`는 조건이 맞지 않을 때 예외를 던지는 헬퍼 메서드다.

단일 프레임 리소스일 경우 `Drawable#invalidateSelf`를 호출해서 다시 그린다.

애니메이션인 경우에는 실행 상태가 아닐 때 실행 상태로 바꾸고

 `state.frameLoader.subscribe(this)`를 호출한 후 다시 그린다.

# Gif의 여러 프레임을 처리하는 GifFrameLoader

`GifDrawable`은 `state: GifState`를 필드로 가지고 있는데 

`GifState`는 `ConstantState`를 상속받는다. 

`ConstantState`를 상속받기 때문에 `Drawble`의 행동과 마찬가지로, 

같은 리소스를 로딩했을 때 여러 `GitDrawable`이 같은 state를 공유할 것이라 예상할 수 있다. 

`state`를 통해 공유하는 것은 `frameLoader: GifFrameLoader`이다.

[**GifFrameLoader**](https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/load/resource/gif/GifFrameLoader.java)의 subscribe는 다음과 같다.

```java
void subscribe(FrameCallback frameCallback) {
  if (isCleared) {
    throw new IllegalStateException("Cannot subscribe to a cleared frame loader");
  }
  if (callbacks.contains(frameCallback)) {
    throw new IllegalStateException("Cannot subscribe twice in a row");
  }
  boolean start = callbacks.isEmpty();
  callbacks.add(frameCallback);
  if (start) {
    start();
  }
}
```

이미 `clear`되었으면 더 이상 진행할 수 없고 동일한 `GifDrawable`은 등록할 수 없다.

`callbacks`는 `GifDrawable`의 리스트인데 우리의 `GifDrawable`을 등록하고, 첫번째 `GifDrawable`이면 `start`를 호출한다.

```java
private void start() {
  if (isRunning) {
    return;
  }
  isRunning = true;
  isCleared = false;

  loadNextFrame();
}
```

# 다음 프레임 처리하기 loadNextFrame

이미 러닝 상태면 종료하고 러닝 상태가 아니면 러닝 상태로 바꾼다. 

여기에서 핵심은 `loadNextFrame`을 호출하는 것이다.

```java
private void loadNextFrame() {
  if (!isRunning || isLoadPending) { // (1)
    return;
  }
  if (startFromFirstFrame) { // (2)
    Preconditions.checkArgument(
        pendingTarget == null, "Pending target must be null when starting from the first frame");
    gifDecoder.resetFrameIndex();
    startFromFirstFrame = false;
  }
  if (pendingTarget != null) { // (3)
    DelayTarget temp = pendingTarget;
    pendingTarget = null;
    onFrameReady(temp);
    return;
  }
  isLoadPending = true; // (4)
  int delay = gifDecoder.getNextDelay();
  long targetTime = SystemClock.uptimeMillis() + delay;

  gifDecoder.advance();
  next = new DelayTarget(handler, gifDecoder.getCurrentFrameIndex(), targetTime);
  requestBuilder.apply(signatureOf(getFrameSignature())).load(gifDecoder).into(next);
}
```

1. 실행 중이 아니거나 로딩이 아직 끝나지 않았다면 진행하지 않는다.
2. 첫 프레임 부터 시작하라는 지시를 받았다면 `pendingTarget` (처리해야 하는 타겟)이 없는 경우인지 확인하고, 디코더를 -1 번째 프레임으로 돌린다. 이렇게 -1로 돌리는 이유는 아래 호출할 `gifDecoder.advance()`가 프레임을 더해 0 번째 프레임으로 이동하기 때문이다. `startFromFirstFrame`을 처리했기 때문에 리셋한다.
3. `pendingTarget` (처리해야 하는 타겟)이 있다면 이를 `onFrameReady`메서드를 통해 처리한다.
4. 이제 (다음 프레임) 로딩 중이다고 체크하고 디코드를 다음 프레임으로 진행시킨다. `requestBuilder.apply`를 통해 비동기로 로딩을 진행시키고 로딩의 상황을 `DelayTarget`이 수행하도록 구성한다.
