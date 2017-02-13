---
layout: post
title: 'Jekyll&github blog에서 latex 문법 사용하기'
tags: [jekyll]
description: >
  Jekyll&github blog에서 latex 문법 사용하기
---

Markdown을 이용하여 글을 작성하는 것은 직관적이면서도 편리한 방법이다. 하지만 markdown만으로는 수학 수식을 입력하기 쉽지 않기 때문에 다양한 플러그인을 사용해야 한다. 그중에서 latex을 사용하기 위한 방법을 소개한다.

자세한 내용은 [여기](http://www.dianacai.com/blog/2015/09/12/making-blog/)를 참고.

## MathJax

`_includes/head.html` 파일에 `MathJax`를 추가해 줌으로써 latex을 이용하여 수학 공식을 작성할 수 있다. `head.html`파일의 맨 위쪽에 다음의 code를 추가한다.

```
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
```

그리고 Matxjax가 정상적으로 동작하기 위해서는 `_config.yml`파일에서 `markdown: kramdown` 으로 되어 있어야 한다. 기본적으로 github & jekyll 에서는 기본적으로 kramdown으로 되어 있을 것이다. 


