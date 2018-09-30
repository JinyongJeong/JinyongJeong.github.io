---
layout: post
title: '[Android] Android  configChanges option (안드로이드 화면 회전시 view 유지방법)'
tags: [android]
description: >
    How to maintain view when screen direction change using configChanges option
---

안드로이드의 경우 화면이 회전되었을 경우 화면을 종료시키고 새로운 layout으로 재 시작한다. 

즉 세로화면 -> 가로화면으로 전환 시  `onDestroy()` 함수가 호출되고 가로모드에서 다시 `onCreate()` 함수가 호출된다.

즉 환경변화가 일어날 경우 기본적인 동작은 activity 의 재시작이다.

이럴때 `AndroidManifest.xml` 파일에 ` android:configChanges`을 설정함으로써 activity가 reset되는것을 막을 수 있다.

이런경우 activity의 `onDestroy()`와 `onCreate()` 함수 대신에 `onConfigurationChanged()`함수가 호출된다.

이러한 옵션이 왜 필요한지 예를 들어보자.

사용자의 이름을 입력하는 Edittext를 1개 포함하는 화면이 있다고 하자.

화면이 처음 구성되는 `onCreate()`함수에서 사용자의 이름변수를 null로 초기화 하고, 화면이 구성된 이후에 사용자가 이름을 입력하였다. 

만약 이때 화면이 가로모드에서 세로모드로 변경되면 화면이 종료되고 다시 시작되기 때문에 사용자의 이름변수는 다시 null이 입력되어 있게 된다. 

이런 상황에서 원하는 동작은 사용자가 이름을 입력하였기 때문에 Edittext는 사용자가 입력된 내용을 그대로 유지하고 있어야 한다. 이럴때 `configChanges` 옵션중에서 `orientation` 옵션을 주면 화면의 방향이 변경되어도 초기화 되지 않는다.

적용시키는 방법은 다음과 같다.

## 1. AndroidManifest.xml 수정
 
 `AndroidManifest.xml` 파일에 `android:configChanges="orientation"` 을 추가해준다. activity안에 넣어주어야 한다. 다른 설정과 함께 작성하면 다음과 같다. 
 
 ```
 <activity
     android:name=".MainActivity"
     android:label="@string/app_name"
     android:windowSoftInputMode="stateAlwaysHidden"
     android:configChanges="orientation"
     android:theme="@style/AppTheme.NoActionBar">
 </activity>
 ```

위의 옵션은 화면의 방향이 변해도 화면이 초기화 되지 않도록 하며 (`android:configChanges="orientation"`), fragment가 변경되었을 때 항상 soft 키보드를 숨기는 옵션 (`android:windowSoftInputMode="stateAlwaysHidden"`) 을 적용하였다. 키보드 관련된 내용은 [다음 글](http://jinyongjeong.github.io/2018/09/30/softkeyboard_hide/)을 참고하면 된다.

## 2. onConfigurationChanged() 함수 작성

1번과 같이 옵션을 추가하고 화면의 방향이 변경 시, `onConfigurationChanged()` 함수가 호출된다. 만약 화면의 방향이 전환되었을 때 수행해야 하는 작업이 있는 경우 이 함수에 추가하면 된다. 

## 3. configChanges의 여러가지 옵션들


* "mcc": SIM 이 Detect 되고 MCC 가 Update 될 경우. ( IMSI Mobile Country Code가 변했을 때 )   
* "mnc": SIM 이 Detect 되고 MNC 가 Update 될 경우. ( IMSI Mobile Network Code가 변했을 때 )
* "locale":  User 가 새로운 Language 를 선택했을 때 ( Locale 이 변경될 때 )
* "touchscreen": Touch Screen Hardware 가 바뀌었을 때 ( 보통은 절대 일어나지 않는 Case 임 )   
* "keyboard": User 가 External Keyboard를 꽂았을 때를 비롯하여 Keyboard 의 Type 변경시
* "keyboardHidden": User 가 Hardware Keyboard를 보이고 감추는 등의 Keyboard의 Accessibility가 변경되었을 때   
* "navigation": Navigation Type ( 트랙볼 / DPad ) 가 변경되었을 때 ( 보통 절대 일어나지 않는 Case )
* "orientation": User가 Device 를 돌리는 등의 행위로 Screen 의 Orientation 이 변경되었을 경우
* "screenLayout": Screen의 Layout이 변했을 때, 다른 Display 가 Activate 되었을 경우 Layout이 변한 경우 
* "fontScale": User 가 새로운 Font Size 를 선택했을 때.
* "uiMode": User 가 Device 를 Desk 나 Car Dock 등에 비치하여 Interface Mode 를 바꾸었을 때


참고사이트: [https://aroundck.tistory.com/36](https://aroundck.tistory.com/36)

위와 같이 다양한 옵션을 사용할 수 있다. 위의 옵션은 or 구분자(`|`)를 통해 다음과 같이 중복 입력 가능하다 

```
android:configChanges="orientation|screenLayout"
```
