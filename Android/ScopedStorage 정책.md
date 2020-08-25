## Intro

사용자 데이터를 보호하고 앱이 사용하는 공간을 줄이기 위해 

Android 10은 External Storage Permission에 대한 권한 동작을 변경했습니다.

Android 11 Developer Preview는 이러한 노력을 계속하면서 개발자가 변경 사항에 적응할 수 있도록 개선 된 기능을 추가합니다.

Google Play에 게시된 대부분의 앱들은 SD카드에 파일을 저장하거나 미디어 파일을 읽는 이유 때문에 저장 권한을 요청을 합니다.

이러한 앱은 앱을 제거한 후에도 남아있는 많은 파일을 디스크에 저장할 수 있습니다. 

또한 다른 앱에서 잠재적으로 민감한 파일을 읽을 수도 있습니다.

Android 10에서는 저장소 권한의 작동 방식을 조정하여 앱에 필요한 액세스 권한을 부여했습니다. 

또한 앱을 삭제할 때 지워지는 지정된 디렉토리에 파일을 저장하도록 앱을 장려함으로써 

파일 혼란을 제한합니다.

Android 10의 스토리지 변경은 세 가지 기본 원칙을 따릅니다.

- **더 나은 기여** : 시스템은 어떤 파일이 어떤 앱에 속하는지 알기 때문에 사용자가 파일을 보다 쉽게 관리 할 수 있습니다.

    또한 앱을 제거 할 때 사용자가 원하지 않는 한 앱이 만든 콘텐츠도 같이 삭제 됩니다.

- **앱 데이터 보호** : 앱이 앱 전용 파일을 외부 저장소에 쓸 때 이러한 파일은 다른 앱에 표시되지 않아야합니다.

- **사용자 데이터 보호** : 사용자가 민감한 전자 메일 첨부 파일과 같은 파일을 다운로드 할 때 대부분의 앱에서 이러한 파일을 볼 수 없습니다

Android 10을 대상으로하는 앱은 저장 권한을 요청하지 않고도 자체 외부 앱 디렉토리에 기여하고 미디어 컬렉션 (오디오, 비디오, 이미지 및 다운로드)을 구성 할 수 있습니다.

저장 권한은 앱이 오디오, 비디오 및 이미지 모음에서 다른 앱이 공유하는 미디어 파일 만 읽을 수 있도록 허용합니다.

그러나 앱이 만들지 않은 Downloads 컬렉션의 파일에 대한 읽기 권한은 부여하지 않습니다.

앱이 Android 10의 다른 앱에서 만든 비 미디어 파일에 액세스하는 유일한 방법은 

Storage Access Framework에서 제공하는 문서 선택기를 사용하는 것입니다.

Android 11에서는 다음 변경 사항을 도입하여 Scoped Storage와 관련된 개발자 경험을 계속 향상시킬 것입니다.

## Media store improvements

Android 10에서는 모든 앱이 MediaStore API를 사용하여 사진, 비디오 및 음악 파일에 액세스해야 했습니다.(권장사항)

그러나 다른 많은 앱들이 File Descriptor 를 사용하여 전환 할 수 없는 File Api에 의존하고 있습니다.

따라서 Android 11에서는 파일 경로를 사용하는 API 및 라이브러리가 다시 활성화 됩니다.

앱은 `requestLegacyExternalStorage` 매니페스트 속성을 사용하여 

Android 10 을 실행하는 사용자의 호환성을 보장합니다.

[https://developer.android.com/preview/privacy/storage#media-files-raw-paths](https://developer.android.com/preview/privacy/storage#media-files-raw-paths)

내부적으로 파일 경로를 사용하는 I/O 요청은 MediaStore API에 위임됩니다.

이 리다이렉션은 격리된 저장소 외부에서 파일을 읽는 성능에 영향을 미칩니다.

또한 파일 경로를 사용한다고 해서 MediaStore API에 대한 이점이 제공되지 않으므로 

MediaStore를 직접 사용하는 것이 좋습니다.

Android 10에서 사용자는 앱이 편집 또는 삭제를 요청한 각 파일을 확인해야 했습니다.

Android 11을 사용하면 한 번에 여러 미디어 파일을 수정하거나 삭제하도록 요청 할 수 있습니다.

시스템 갤러리 앱은 이러한 대화 상자를 표시하지 않습니다.

[https://miro.medium.com/max/619/0*oyE3jpmd6l50oexF](https://miro.medium.com/max/619/0*oyE3jpmd6l50oexF)

## Storage Access Framework changes

스토리지의 접근 권한을 제한한 후 여러 개발자들이 SAF를 통해 파일 시스템을 탐색 할 수 있습니다.

그러나 SAF는 공유 스토리지에 대해서는 적합하지 않습니다.

따라서 특정 경로에 대한 가시성을 제한하도록 업데이트 했습니다.

[Storage updates in Android 11 | Android Developers](https://developer.android.com/preview/privacy/storage#file-directory-restrictions)

Android 11 에서 사용자는 

- Root 다운로드 디렉터리,
- 신뢰할 수 있는 각 SD 카드 볼륨의 루트 디렉터리
- 외부 앱 디렉터리

에 대한 디렉터리 액세스 권한을 부여 할 수 있습니다.

앱은 SAF API 및 문서 선택기를 사용하여 사용자에게 공유 저장소에서

개별 파일을 선택하도록 요청 할 수 있습니다.

## 파일 관리자 앱에 대한 특별한 권한

파일관리자 또는 백업 앱과 같이 공유 저장소에 대한 광범위한 액세스가 필요한 앱의 경우

Android 11은 `MANAGE_EXTERNAL_STORAGE` 라는 특수 권한을 도입합니다.

이렇게 하면 미디어가 아닌 파일을 포함하여 모든 공유 저장소에 대한 읽기 및 쓰기 액세스 권한이 부여 됩니다.

그러나 내부 및 외부 저장소 내에서 앱별 디렉터리의 콘텐츠에 액세스할 수 없습니다.

실제로 필요한 경우 일부 앱이 외부 저장소의 파일에 액세스 할 수 있도록 계속 허용하고 싶습니다.

Android11 에서 이러한 앱은 `MANAGE_EXTERNAL_STORAGE`를 선언하여 사용자가 설정에서 

'**모든 파일 액세스**' 를 허용하도록 요청할 수 있습니다.

다음은 허용되는 몇가지 앱 예입니다.

- 파일 매니저 - 사용자가 파일을 관리할 수 있도록 하는 것이 주 목적인 앱
- 백업 및 복원 - 파일에 대한 대량 액세스가 필요한 앱 (예 : 장치 전환 또는 클라우드에 백업)

그러나 앱에 워드프로세서 앱과 같은 단일 파일에 대한 액세스가 필요한 경우 SAF를 사용해야 합니다.

여기에서 권한에 관한 Google Play 정책을 확인 할 수 있습니다.

[Use of All files access (MANAGE_EXTERNAL_STORAGE) permission](https://support.google.com/googleplay/android-developer/answer/9956427)

Android 11에서 앱에 `MANAGE_EXTERNAL_STORAGE` 가 필요하거나 

파일 경로에 의존하는 API를 사용하는 경우 Android 메니페스트에서 

```kotlin
requestLegacyExternalStorage = true
```

를 선언하여 이전버전과의 호환성을 지원할 수 있습니다.

## 참조

[Storage updates in Android 11 | Android Developers](https://developer.android.com/preview/privacy/storage)

[Preparing for Scoped Storage (Android Dev Summit '19)](https://youtu.be/UnJ3amzJM94)

[https://youtu.be/UnJ3amzJM94](https://youtu.be/UnJ3amzJM94)
