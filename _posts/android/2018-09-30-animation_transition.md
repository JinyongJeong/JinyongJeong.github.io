---
layout: post
title: '[Android] Android fragment transition with animation (안드로이드 프레그먼트 애니메이션 화면전환)'
tags: [android]
description: >
    Fragment transition with animation
---

이 글에서는 fragment의 화면전환 할 때 애니메이션을 넣는 방법에 대해서 설명한다. 

## 1. 애니메이션 정의

화면전환에 대한 애니메이션을 각각 정의해 주어야 한다. 

애니메이션을 정의하는 경로는 `res/anim/파일명.xml`이다 .

다양한 동작을 하는 애니메이션들을 정의해보자

### `enter_from_left.xml`

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"  >
    <objectAnimator
    android:propertyName="x"
    android:duration="600"
    android:valueFrom="-400dp"
    android:valueTo="0"
    android:valueType="floatType" />
</set>
```

### `enter_from_right.xml`

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android" >
    <objectAnimator
    android:duration="600"
    android:propertyName="x"
    android:valueFrom="400dp"
    android:valueTo="0"
    android:valueType="floatType" />
</set>
```

### `exit_to_left.xml`

```
<?xml version="1.0" encoding="utf-8"?>
    <set xmlns:android="http://schemas.android.com/apk/res/android" >
    <objectAnimator
    android:duration="600"
    android:propertyName="x"
    android:valueFrom="0"
    android:valueTo="-400dp"
    android:valueType="floatType" />
</set>
```
### `exit_to_left.xml`

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <objectAnimator
    android:duration="600"
    android:propertyName="x"
    android:valueFrom="0"
    android:valueTo="400dp"
    android:valueType="floatType" />
</set>
```

모든 동작은 좌우로 fragment를 이동하는 애니메이션이며  `property`의 이름은 `x` 이다. 

이동하는 애니메이션 뿐만 아니라 `fade_in`, `fade_out` 애니메이션도 다음과 같이 정의할 수 있다.


### `fade_in.xml`

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"  >
    <objectAnimator
    android:duration="600"
    android:propertyName="alpha"
    android:valueFrom="0"
    android:valueTo="1"
    android:valueType="floatType"
    />
</set>
```

### `fade_out.xml`

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"  >
    <objectAnimator
    android:duration="600"
    android:propertyName="alpha"
    android:valueFrom="1"
    android:valueTo="0"
    android:valueType="floatType"
    />
</set>
```

## 2. 애니메이션을 통한 fragment 전환

애니메이션을 적용한 fragment의 전환을 위해서는 기본적은 fragment transition에 `setCustomAnimations` 옵션을 추가해주면 된다. 다음은 이전 화면은 왼쪽으로 나가면서 오른쪽에서 새로운 fragment가 들어오는 애니메이션이 적용되는 예이다. 

```
FragmentManager manager = getFragmentManager();
manager.beginTransaction()
    .setCustomAnimations(R.anim.enter_from_right, R.anim.exit_to_left, R.anim.enter_from_left,R.anim.exit_to_right)
    .replace(R.id.content_main, fragment)
    .commit();
```

위의 예제는 뒤로가기를 눌렀을 때 이전 fragment로 가지 않도록 stack에 누적하지 않는 코드이다. 만약 뒤로가기를 눌렀을 때 다시 이전 화면으로 돌아가는 코드로 만들고 싶다면 `transaction.addToBackStack(null)` 를 추가해주면 된다. 

`setCustomAnimations`의 인자는 총 4개이다. 

첫번째 인자는 fragment의 transition이 수행되었을 때 새로운 fragment의 애니메이션이며 두번째 인자는 기존에 화면에 있던 fragment의 애니메이션이다. 

즉 새로운 fragment는 오른쪽에서 들어오면서 기존에 있던 fragment는 왼쪽으로 나간다. 

세번째 인자는 뒤로가기를 눌렀을 때, 즉 이전화면으로 이동 시의 애니메이션이다. 

즉 stack에 쌓여있던 fragment는 왼쪽에서 들어오며, 기존에 있던 fragment는 오른쪽으로 사라진다. 


