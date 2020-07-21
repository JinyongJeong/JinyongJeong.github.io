---
layout: post
title: 'Background로 실행 중인 프로세스 output 확인'
tags: [ubuntu]
comments: true
description: >
  Check stdout of running process
sitemap :
  changefreq: weekly
  priority : 1.0
---
`top`로 확인 가능한 process의 terminal output를 확인하기 위한 명령어

```bash
# std out
cat /proc/{pid of process}/fd/1
# std err
cat /proc/{pid of process}/fd/2
```