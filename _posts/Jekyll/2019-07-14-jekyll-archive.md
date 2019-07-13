---
layout: post
title:  "jekyll-archives 플러그인 오류"
date:   2019-07-14 02:40:00 +0900
author: Gnues
categories: jekyll
---

깃헙 페이지에 Jekyll 외부 플러그인을 사용할 수 있게 되었으나 `jekyll-archives` 기능이 제대로 작동하지 않는 문제가 생겼다.

현재 블로그는 [bencentra/centrarium](bencentra/centrarium) 테마를 기반으로 제작하고 있는데, 기본적으로 `jekyll-archives`를 미리 사용하고 있다.

그런데, 각 포스트(작성글)에서 사용하는 카테고리, 태그 등의 아카이브 요소들을 기준으로 검색하는 페이지가 간헐적으로 작동하지 않는다.

각 포스트의 `category`, `tag`를 설정하는 것과 [posts 페이지](https://gnueskob.github.io/posts/)에서의 포스트 별 카테고리 랜더링까지도 문제가 없다.

하지만 `category`와 `tag`를 포함하는 포스트 글을 묶어 보여주는 페이지가 이상하게도 404 에러가 뜬다.

로컬에서는 위의 제한사항 없이도 정상적으로 작동하지만 깃헙 페이지로 호스팅할 경우에만 나타난다.

***

## **해결**

직접 정적파일을 트래킹해기도 하고 카테고리, 태그 전부 바꿔가며 테스트를 해보았는데 결국 대소문자 문제였다.

`jekyll-archives`로 특정 요소를 enabled하여 사용하는 경우 `lower-case`를 사용해야 함을 명심하자...

```yml
# 현재 지킬 아카이브 설정
# Archive settings
jekyll-archives:
  enabled:
    - categories
    - tags
  layout: 'archive'
  permalinks:
    category: '/category/:name/'
    tag: '/tag/:name/'
```

- [jekyll-archives Github](https://github.com/jekyll/jekyll-archives/)
