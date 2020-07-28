---
layout: post
title: '[git] remote에서 삭제된 local branch 삭제'
tags: [software]
comments: true
description: >
  Delete local branch that was deleted on remote
sitemap :
  changefreq: weekly
  priority : 1.0
---
Master로 merge가 완료된 branch는 remote에서 삭제된다. 하지만 local 에서는 이러한 branch가 남아 있기 때문에 `checkout` 으로 branch간 이동을 할 때 불편하다. 다음 명령어로 remote에서 삭제된 local branch를 삭제할 수 있다. 

```bash
git branch -vv | grep ': gone]'|  grep -v "\*" | awk '{ print $$1; }' | xargs -r git branch -d
```