---
layout: post
title: '[Android] Android soft keyboard option (안드로이드 키보드 옵션, 자동 키보드 열림 방지)'
tags: [android]
description: >
    Soft keyboard 옵션
---

이 글에서는 안드로이드의 soft keyboard를 조절하는 옵션에 대해서 설명한다. 

만약 한 fragment에 edittext가 존재할 때, 그 fragment로 transition이 수행되면 자동으로 edittext에 입력을 할 수 있도록 keyboard 가 열린다. 

하지만 사용자가 입력을 원하는 순간에만 키보드가 열리게 할 필요가 있다 이럴때 soft keyboard에 대한 옵션을 조절하면 된다.

옵션을 조절하기 위해서는 다음과 같이 옵션을 추가해준다. 

## 1. `AndroidManifest.xml`에 옵션추가

`AndroidManifest.xml`의 activity옵션에 다음과 같은 옵션을 추가해준다. 

```
android:windowSoftInputMode="stateAlwaysHidden"
```

즉 전체 activity에 대한 설정을 보면

```
<activity
    android:name=".MainActivity"
    android:label="@string/app_name"
    android:windowSoftInputMode="stateAlwaysHidden"
    android:configChanges="orientation|screenSize"
    android:theme="@style/AppTheme.NoActionBar">
</activity>
```

위와 같이 된다. 

## 2. 옵션에 대한 설명

* stateUnspecified: 키보드의 디폴트 설정 값
* stateUnchanged: 키보드의 마지막 상태로 유지
* stateHidden: activity 실행 시 키보드가 자동으로 올라오는 것을 방지
* stateAlwaysHidden: activity 실행 시 항상 자동으로 올라오는 것을 방지
* stateVisible: activity 실행 시 키보드가 자동으로 올라옴
* stateAlwaysVisible: activity 실행 시 항상 키보드가 자동으로 올라옴
* adjustUnspecified: 시스템이 알아서 상황에 맞는 옵션을 설정
* adjustResize: 키보드가 올라와도 edittext가 UI 화면에 보이도록 activity를 resize
* adjustPan: 키보드가 올라오면 edittext에 맞춰 화면 UI가 실종됨 (위아래 잘림)


개인적으로 fragment전환 시 키보드가 올라오는 것을 방지하기 위해 `stateAlwaysHidden` 옵션을 사용하였다.
