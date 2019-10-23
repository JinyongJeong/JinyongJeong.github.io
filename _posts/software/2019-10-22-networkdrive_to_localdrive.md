---
layout: post
title: 'NAS의 Network Drive를 Local Drive로 인식시키기'
tags: [software]
description: >
  NAS의 Network Drive를 Local Drive로 인식시키기
---

[참고](https://icesquare.com/wordpress/lightroom-how-to-save-lightroom-catalog-on-network-drive-on-windows/)


## Network drive 를 local drive로 인식시키기



라이트룸으로 사진을 관리할 때 Catalog를 이용한다. NAS를 이용하는 경우 사진파일은 Network drive로 관리가능하지만 catalog는 network drive에 저장할 수 없다. (라이트룸 정책상)

따라서 일반적으로 NAS를 사용할 때 network drive로 사용하기 때문에 1개의 카탈로그를 만들어서 사용할 경우에는 dropbox 혹은 시놀로지의 cloud station을 이용해서 catalog를 컴퓨터의 local에 동기화하여 사용한다. 하지만 프로젝트별로 카탈로그를 따로 관리하고 싶을 때는 난감하다. 이럴때는 network drive를 local drive로 인식시키면 된다.


윈도우에서 `cmd`창을 열고 다음과 같이 입력.

```
subst X: \\servername\foldername
```

앞의 X: 는 로컬 드라이브의 이름, 뒤에는 network drive의 서버이름이다. 나의 경우는

```
subst X: \\jinyong-data\photo
```

로 입력하였다.

이렇게 입력하고 내컴퓨터로 이동하면 네트워크 위치에 드라이브가 생성되어 있을 것이다. 연결이 끊긴 네트워크 드라이브라고 나올테지만 문제없이 접속이 될 것이다. 이렇게 한 후에 라이트룸에서 카탈로그를 새로 생성된 드라이브에 저장을 해보면 문제없이 저장 될 것이다. 

그다음에는 컴퓨터가 로그인 될 때마다 자동으로 드라이브를 생성하도록 스케쥴러를 작성해야 한다. 

프로그램 시작에서 `task scheduler` 혹은 `작업 스케쥴러`를 검색해서 실행한다.

오른쪽 창에서 `작업 만들기`를 클릭한 후 

`트리거` 창에서 `새로만들기` 후 작업시작을 `로그온 할 때`로 하고 확인을 누른다. 

`동작`창에서 `새로만들기` 후 동작은 `프로그램 시작` 스크립트 창에 `subst X: \\servername\foldername`를 입력하고 확인을 눌러준다.

이렇게 한 후에 컴퓨터를 재부팅 하면 자동으로 네트워크 드라이브가 로컬드라이브로 잡힐 것이다.



