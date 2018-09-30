---
layout: post
title: '[Android] Android export file in internal storage (안드로이드 내부 저장소의 데이터 출력, fileprovider)'
tags: [android]
description: >
    export file in internal storage (내부 저장소의 데이터 출력, fileprovider)
---

안드로이드에서는 보안을 위해서 데이터를 내부에 저장하는 것이 가장 안전한 방법이다. 

안드로이드의 데이터 저장위치에 대한 내용을 검색해 보면 크게 

내부저장소, 외부저장소 (공유영역, 어플고유영역)으로 나뉘어 진다.

내부저장소는 어플 자체에서만 접근 가능하며, 사용자도 접근하지 못한다. 

외부저장소의 공유영역은 모든 어플에서 접근 가능하며, 어플을 삭제해도 데이터가 남는 영역이다. 

외부저장소의 고유영역은 모든 어플에서 접근 가능하지만, 어플을 삭제할때 데이터도 같이 삭제되는 영역이다.

이러한 각 데이터 저장 영역에 대해서는 많은 포스팅이 있으니 참고하자.

[http://jinyongjeong.github.io/2018/09/30/android_path/](http://jinyongjeong.github.io/2018/09/30/android_path/)

[http://ismydream.tistory.com/124](http://ismydream.tistory.com/124)

[http://bitsoul.tistory.com/117](http://bitsoul.tistory.com/117)

[https://developer88.tistory.com/28](https://developer88.tistory.com/28)

만약 보안을 위해서 내부영역에 데이터를 저장했는데 이메일을 통해서 데이터를 출력하고 싶다면 어떻게 해야할까.

가장 쉽게 생각할 수 있는것이 외부영역으로 데이터를 복사한 후에 데이터를 출력하고 삭제하는 방법이다. 

하지만 이럴때 사용할 수 있는 것이 `fileprovider`이다. 

`fileprovider`는 내부 영역에 특정 파일을 외부 어플에서 접근 가능하도록 제공해 주는 역할을 하며

특정 파일을 지정해서 공유할 수 있기 때문에 안전하다. 

`fileprovider`를 사용하기 위해서는 다음과 같은 절차가 필요하다. 


## 1. Manifest 설정

`AndroidManifest.xml` 의 `application` 안쪽에 선언

```
<provider
android:name="android.support.v4.content.FileProvider"
android:authorities="${applicationId}.fileprovider"
android:grantUriPermissions="true"
android:exported="false">
<meta-data
android:name="android.support.FILE_PROVIDER_PATHS"
android:resource="@xml/file_path" />
</provider>
```

## 2. provider의 경로 설정

`res/xml/file_path.xml` 을 생성하고 다음을 입력

```
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <paths>
        <files-path path="/" name="default" />
    </paths>
</paths>
```

경로가 `xml` 폴더가 아닐경우 오류가 난다


## 3. Email을 통해 데이터 출력

```
File file = new File(activity.getFilesDir(), "/exported_data.zip");
Uri uri = FileProvider.getUriForFile(context, activity.getPackageName()+".fileprovider", file);
Intent i = new Intent(Intent.ACTION_SEND);
i.setFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
i.setType("message/rfc822");
i.putExtra(Intent.EXTRA_SUBJECT, "Gaitspeedometer data");
i.putExtra(Intent.EXTRA_STREAM, uri);

try {
    startActivity(Intent.createChooser(i, "Send file..."));
} catch (Exception e) {
    Toast.makeText(activity, "No activity found for export", Toast.LENGTH_SHORT).show();
}
```

위와 같이 코드를 실행하면 내부 영역에 있는 `exported_data.zip`파일이 이메일에 첨부되고 출력할 수 있게 된다.


