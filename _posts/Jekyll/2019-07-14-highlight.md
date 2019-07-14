---
layout: post
title:  "jekyll에서 highlight.js 적용"
date:   2019-07-14 13:00:00 +0900
author: Gnues
categories: jekyll
tags:	jekyll highlight.js
---

## highlight.js 적용

Jekyll은 기본적으로 [highlight.js](https://highlightjs.org/)를 지원한다.

페이지에 `highlight.js`를 적용하기 위해 원하는 버전의 `css`, `js`를 받아와야 한다.

현재 최신 버전은 9.15.8 버전이다.

```html
<link rel="stylesheet"
      href="//cdnjs.cloudflare.com/ajax/libs/highlight.js/9.15.8/styles/default.min.css">
<script src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/9.15.8/highlight.min.js"></script>
```

Jekyll 프로젝트를 빌드하면 `markdown`의 `Fenced Code Block`이나 `Liquid`의 `{% raw %}{% highlight <language> %}{% endraw %}`의 코드가 자동적으로 html `<code>` 태그로 변경된다.

이 때 Jekyll이 자동으로 highlight.js 기능을 사용할 수 있도록 `attribute`를 설정해준다.

## 테마에서 기본으로 지원하는가

현재 사용하는 [bencentra/centrarium](bencentra/centrarium) 테마에서는 `highlight.js`기능을 이미 사용중이다.

그런데, 간혹 적용이 되지 않거나 원하는 highlight 테마적용 자체가 되지않아 원인을 분석해보았다.

테마 코드를 까보니... 바꾸고 싶어했던 highlight 테마를 지원하지 않는 버전을 사용중이었다.

```html
<!-- head.html -->
<link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/highlight.js/9.9.0/styles/{{ site.highlightjs_theme }}.min.css">
...

<!-- footer.html -->
<script src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/9.9.0/highlight.min.js"></script>
```

9.9.0 버전으로 하드코딩 되어있었기에 높은 버전에서 지원하는 style이나 언어를 지원하지 않았다.

때문에 현재 버전을 최신으로 바꾸는 작업이 필요했고, 유지보수를 편하게 하기 위해 버전 관리를 한 군데서 모아서 처리하도록 바꾸었다.

```html
<!-- head.html -->
<link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/highlight.js/{% raw %}{{ site.highlightjs_version }}{% endraw %}/styles/{% raw %}{{ site.highlightjs_theme }}{% endraw %}.min.css">
...

<!-- footer.html -->
<script src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/{% raw %}{{ site.highlightjs_version }}{% endraw %}/highlight.min.js"></script>
```

기존에 `highlight.js`를 받아오는 코드에서 version을 `_config.yml`에서 처리할 수 있도록 위와 같이 변경한다.

```yml
# _config.yml
highlightjs_version: "9.15.8"
highlightjs_theme: "vs2015"
```

설정 파일에서 버전과 테마를 직접 관리한다.
