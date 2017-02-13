---
layout: post
title: 'Ubuntu rsync 오류 및 옵션'
tags: [ubuntu]
description: >
  Ubuntu rsync 오류 및 옵션
---
rsync와 scp는 원격으로 파일을 복사할때 사용하는 명령어이다. 

내가 주로 사용하는 rsync 명령어 및 옵션은 `rsync -avPh source dest'으로 평소에 사용할 때에는 별 문제가 없다. 하지만 exFAT로 format되어있는 하드디스크로 복사를 하려고 할 때 

```
failed to set permissions on 경로 Function not implemented (38)
```

이라는 오류가 뜨면서 재대로 복사가 되지 않는다. rsync exFAT로 검색하여 찾은 해결법은 [다음](http://blog.marcelotmelo.com/linux/ubuntu/rsync-to-an-exfat-partition/)에서 확인할 수 있다.

위의 포스트에 따르면 `-a` 옵션이 문제가 된다. 

`-a` 옵션은 `-archive`의 옵션으로 rsync를 아카이브 모드로 실행을 하며 [-rlptgoD]의 동작을 한다. 즉 7개의 옵션을 모두 포함하고 있는 옵션이란 뜻이다. 

이때 exFAT로 복사할 때 문제가 되는 부분은 permission과 관련되는 `pgo`옵션이다. 따라서 이 3가지 옵션을 제외한 나머지 옵션을 `-a` 대신 넣어주면 된다. 즉 `rsync -rltDvPh` 이렇게 옵션을 넣어주면 `rsync -avPh`에서 위의 3가지 옵션을 제외한 옵션이 된다. 

이렇게 옵션을 바꾸어 주면 문제없이 exFAT format으로도 복사가 잘 된다. 

자세한 rsync 들의 옵션들은 [다음 블로그](http://gyus.me/?p=214)에 잘 정리되어 있다.


참고URL

<http://blog.marcelotmelo.com/linux/ubuntu/rsync-to-an-exfat-partition/>

<http://gyus.me/?p=214>
