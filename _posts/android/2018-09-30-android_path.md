---
layout: post
title: '[Android] Android data path and method (안드로이드 저장소 경로 및 경로 획득 함수)'
tags: [android]
description: >
    Data path and method (저장소 경로 및 경로 획득 함수)
---

이 글은 [이 글](http://egloos.zum.com/pavecho/v/7204359)을 참고하여 작성하였다.

안드로이드 시스템에서 데이터 저장 장소는 크게 2가지로 나뉘어진다. 

첫번째는 내부 저장소로 이 공간은 해당 앱에서만 접근 가능하다. 유저도 이 공간에는 직접 접근할 수 없다. 

두번째는 외부 저장소로 외부저장소는 공용영역과 고유영역으로 나뉘어진다. 

외부저장소의 공용영역은 다양한 어플이 접근할 수 있는 영역이며 어플이 삭제되어도 이 경로에 저장된 데이터는 유지된다.

반면 외부저장소의 고유영역은 다른 어플도 접근 가능하며, 어플이 삭제시 이 공간의 데이터도 삭제된다.

보안을 위해서는 외부 어플이 접근할 수 없도록 내부 저장소에 저장하는 것이 안전하며, 

만약 내부 저장소에 저장된 데이터를 외부 어플로 전달하고 싶은경우 (email 첨부, 다른어플로 열기 등)에는 fileprovider를 사용해야 한다. 

fileprovider를 이용해서 데이터를 전달하는 방법에 대해서는 [다음 글](http://jinyongjeong.github.io/2018/09/29/fileprovider/)을 참고하기 바란다.


## 내부저장소

각 애플리케이션에서만 데이터를 읽고 쓸 수 있다.

### 1. 캐시(Cache)

임시 파일들이 저장된다.
File Context.getCacheDir()
내부 저장소의 캐시 디렉터리 경로를 반환한다.
경로: /data/data/패키지 이름/cache

### 2. 데이터베이스(Database)

데이터베이스 파일들이 저장된다.
File Context.getDatabasePath(String name)
데이터베이스 파일의 경로를 반환. 인자로 데이터베이스 파일의 이름을 넘겨준다.
경로: /data/data/패키지 이름/databases

### 3. 일반 파일

일반 파일이 저장되는 영역이다. 

이 경로는 Context.openFileOutput(String, int)를 사용하여 생성되는 파일이 저장되는 경로와 동일하다.
File Context.getFilesDir()
일반 파일들의 저장 경로를 반환한다.
경로: /data/data/패키지 이름/files

각 일반 파일들의 경로를 가져오기
File Context.getFileStreamPath(String name)
일반 파일이 저장된 공간에서 특정 이름을 가지는 파일의 경로를 반환한다.
인자로 확장자를 포함한 파일 이름을 넘겨준다.
경로: /data/data/패키지 이름/files/파일이름

## 외부저장소 (공용영역)

애플리케이션을 삭제해도 데이터는 남아있다.

### 1. 최상위 경로

외부 저장소(SD카드)의 최상위 경로를 반환한다.
static File Environment.getExternalStorageDirectory()
경로: /mnt/sdcard 또는 /storage/emulated/0 등 기종마다 다르다.   

### 2. 특정 데이터를 저장하는 영역

여러 애플리케이션에서 공용으로 사용할 수 있는 데이터들을 저장하며 데이터의 유형에 따라 별도의 디렉터리를 사용한다.
이 영역에 데이터를 저장하기 전에, 해당 디렉터리가 존재하는지 확인해야 한다. 존재하지 않으면 FileNotFoundException이 발생하기 때문에 File.mkdirs()를 사용하여 없을 경우 새 디렉터리를 생성해 준다. 
다음의 함수를 이용해서 external 디렉터리가 존재하는지, 그리고 write가 가능한지 확인할 수 있다.

```
public static boolean checkAvailable() {

    // Retrieving the external storage state
    String state = Environment.getExternalStorageState();

    // Check if available
    if (Environment.MEDIA_MOUNTED.equals(state)) {
        return true;
    }
    return false;
}

public static boolean checkWritable() {
    // Retrieving the external storage state
    String state = Environment.getExternalStorageState();

    // Check if writable
    if (Environment.MEDIA_MOUNTED.equals(state){
        if(Environment.MEDIA_MOUNTED_READ_ONLY.equals(state)) {
            return false;
        }else{
            return true;
        }
    }
    return false;
}
```
외부 저장소의 해당 경로를 얻기위해서는 다음 함수를 사용한다
static File Environment.getExternalStoragePublicDirectory(String type)
인자로 디렉터리의 유형을 넘겨준다.

[각 인자 및 경로]

* Environment.DIRECTORY_ALARMS : 알람으로 사용할 오디오 파일 저장 (/mnt/sdcard/Alarms)
* Environment.DIRECTORY_DCIM : 카메라로 촬영한 사진 저장 (/mnt/sdcard/DCIM)
*  Environment.DIRECTORY_DOWNLOADS : 다운로드한 파일 저장 (/mnt/sdcard/Download)
* Environment.DIRECTORY_MUSIC : 음악 파일 저장 (/mnt/sdcard/Music)
* Environment.DIRECTORY_MOVIES : 영상 파일 저장 (/mnt/sdcard/Movies)
* Environment.DIRECTORY_NOTIFICATIONS : 알림음으로 사용할 오디오 파일 저장 (/mnt/sdcard/Notifications)
* Environment.DIRECTORY_PICTURES : 그림 파일 저장 (/mnt/sdcard/Pictures)
* Environment.DIRECTORY_PODCASTS : 팟캐스트 파일 저장 (/mnt/sdcard/Podcasts)

## 외부저장소 (고유영역)

외부 저장소 - 애플리케이션 고유 영역
이 영역에 저장된 데이터는 애플리케이션이 삭제될 때 같이 삭제되며 다른 애플리케이션에서 해당 데이터에 접근하는 것이 가능하다.

### 1. 특정 데이터를 저장하는 영역

애플리케이션 고유 영역에도 공용 영역과 마찬가지로 각 데이터 유형별로 데이터를 저장하는 표준 디렉터리를 제공한다.
File Context.getExternalFilesDir(String type)
인자로 디렉터리의 유형을 넘겨준다.

[각 인자 및 경로]

* Environment.DIRECTORY_ALARMS (/mnt/sdcard/Android/data/패키지 이름/files/Alarms)
* Environment.DIRECTORY_DCIM (/mnt/sdcard/Android/data/패키지 이름/files/DCIM)
* Environment.DIRECTORY_DOWNLOADS (/mnt/sdcard/Android/data/패키지 이름/files/Download)
* Environment.DIRECTORY_MUSIC  (/mnt/sdcard/Android/data/패키지 이름/files/Music)
* Environment.DIRECTORY_MOVIES (/mnt/sdcard/Android/data/패키지 이름/files/Movies)
* Environment.DIRECTORY_NOTIFICATIONS (/mnt/sdcard/Android/data/패키지 이름/files/Notifications)
* Environment.DIRECTORY_PICTURES (/mnt/sdcard/Android/data/패키지 이름/files/Pictures)
* Environment.DIRECTORY_PODCASTS (/mnt/sdcard/Android/data/패키지 이름/files/Podcasts)
* null (/mnt/sdcard/Android/data/패키지 이름/files)

### 2. 캐시 데이터를 저장하는 영역

애플리케이션에서 사용하는 임시 데이터를 외부 저장소에 저장한다.
File Context.getExternalCacheDir()
경로: /mnt/sdcard/Android/data/패키지 이름/cache

* 외부 저장소는 사용자 데이터(사진, 음악 등)이 저장되는 영역이다. 일반적으로 단말기의 외장 SD카드를 지칭하지만, 단말기에 따라서는 이 영역이 외장 SD카드가 아닌 단말기 내부에 탑재되어 있는 경우도 있다.(넥서스S) 또는, 단말기 내에 탑재된 외장 메모리 영역 외에 별도의 SD카드도 지원하는 단말기도 있다.(갤럭시S)
