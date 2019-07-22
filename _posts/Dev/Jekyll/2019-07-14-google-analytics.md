---
layout: post
title:  "Google Analytics 적용"
date:   2019-07-14 16:30:00 +0900
author: Gnues
categories: jekyll
tags:	jekyll google google-analytics
---

## Google Analytics

Github page에서는 방문자 통계 기능을 제공하지 않으므로 [구글 애널리틱스](https://analytics.google.com/)를 이용하여 확인할 수 있다.

<https://analytics.google.com/>에서 가입하여 계정을 얻고, 속성을 만들어 블로그 페이지를 등록한다.

## 속성 만들기

블로그 페이지를 구글 애널리틱스에 등록하기 위해서는 **속성**을 추가해야한다.

애널리틱스 페이지의 **관리** 페이지로 들어가 웹사이트 이름, URL, 카테고리, 시간대를 정의 후 추가한다.

성공적으로 페이지에 관한 속성을 추가하면 **추적 ID**를 얻을 수 있다.

`UA-XXXX-1` 형태의 ID를 통해 자신의 웹사이트로 방문하는 기록들을 구글 애널리틱스가 조회할 수 있다.

`_config.yml`에서 추적 ID를 관리하게끔 설정한다.

```yml
...
# Google analytics
ga_tracking_id: "UA-XXX-1"
```

추적 ID를 얻고 아래의 추적 코드를 자신의 웹사이트 `<head>` 태그 부분에 추가한다

```js
<!-- Global site tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id={% raw %}{{ stie.ga_tracking_id }}{% endraw %}"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', '{% raw %}{{ stie.ga_tracking_id }}{% endraw %}');
</script>
```

`{% raw %}{{ stie.ga_tracking_id }}{% endraw %}`부분에 위에서 얻었던 추적 ID가 들어간다.

## 주의

만약 방문자가 Add-block등의 프로그램을 사용한다면, 구글 애널리틱스의 추적 코드가 비활성화되어 동작하지 않게 된다.

스스로 구글 애널리틱스가 제대로 적용되었는지 확인할 때 이 점을 참고해야 한다.
