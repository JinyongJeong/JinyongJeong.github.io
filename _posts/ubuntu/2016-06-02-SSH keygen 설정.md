---
layout: post
title: 'SSH keygen 설정'
tags: [ubuntu]
description: >
  ssh 접속 시 비밀번호를 묻지 않도록 key를 생성하는 방법
---
# 1. ssh 접속 방법

기본적으로 ssh 접속하는 방법은 다음과 같다

```
ssh user@hostname.example.com
```

위의 명령어로 ssh 접속을 했을 경우 접속 시 마다 password를 물어보게 된다. 만약 `.sh` 스크립트를 작성해서 어떤 동작을 시키려고 하거나 매번 접속할 때마다 password를 입력하는 것이 번거로울 때 ssh key를 생성하여 등록해 놓으면 매번 입력할 필요가 없다. 

# 2. ssh key 생성

ssh key를 생성하기 위해서는 다음의 명령어를 입력한다

```
ssh-keygen
```

명령을 입력 후 파일 명 및 몇가지 질문을 할 것인데 모두 디폴트로 설정하고 넘어가면 `/home/username/.ssh/id_rsa` 파일이 생성된다. 위의 파일을 열어보면 ssh의 key가 생성된 것을 확인할 수 있다.

# 3. ssh key copy

이렇게 생성된 ssh key를 ssh 서버에 복사해야 한다.

```
ssh-copy-id user@hostname.example.com
```

copy명령어를 입력하면 password를 물어볼 것인데, 이때 정확한 password를 입력하면 ssh key가 서버로 복사된다. 복사가 정상적으로 이루어 졌다면, 다음 접속부터는 비밀번호를 다시 물어보지 않을 것이다. 

# 4. "Agent admitted failure to sign using the key" Error

만약 ssh key 등록 후 ssh 접속 시 `Agent admitted failure to sign using the key` 의 error가 발생하며 password를 물어볼 경우 `ssh-add` 명령어를 통해 ssh key를 ssh agent로 load 함으로써 고칠 수 있다. 


