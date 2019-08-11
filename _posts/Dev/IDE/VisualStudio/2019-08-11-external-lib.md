---
layout: post
title:  "Visual Studio 2019 - External Library import"
date:   2019-08-10 21:00:00 +0900
author: Gnues
categories: editor
tags:	editor visual-studio external-library
---

이 글에선 Visual Studio에서 외부 라이브러리를 사용하는 방법을 소개한다.

Visual Studio 2019를 기준으로 작성되었다.

[![img][ext-lib-boost]{: style="max-width: 70%; height: auto;" }][ext-lib-boost]{: data-lightbox="falcon9-large" data-title="boost" }

***

## 프로젝트 설정

현재 열려있는 솔루션의 프로젝트 중 외부 라이브러리를 사용할 프로젝트의 설정을 조정해야 한다.

솔루션 탐색기에서 원하는 프로젝트에 커서 우클릭으로 설정 창을 열거나 `Alt + Enter` 단축키를 이용한다.

[![img][proj-config]{: style="max-width: 60%; height: auto;" }][proj-config]{: data-lightbox="falcon9-large" data-title="config" }

***

## 추가 포함 디렉터리

`#include`를 통해 라이브러리를 사용하기 위해 프로젝트에 라이브러리의 헤더를 포함시켜야 한다.

프로젝트의 설정 창을 열었다면, `C/C++` 탭에서 **추가 포함 디렉터리**를 수정해야 한다.

> `C/C++`탭이 없다면, 현재 프로젝트에 일단 `.cpp`파일 하나를 생성해보자.

라이브러리의 각종 헤더파일 등이 위치한 폴더 경로를 등록한다.

[![img][additional-include-dir]][additional-include-dir]{: data-lightbox="falcon9-large" data-title="additional-include-dir" }

추가 포함 디렉터리 우측 입력란의 화살표(∨), `<편집...>`을 클릭하여 여러가지 디렉터리 경로를 추가할 수 있다.

위 이미지의 예시는 `boost`라이브러리를 추가한 상태이다.

예시로 볼 수 있듯이 경로는 절대경로, 상대경로 모두 가능하며 비주얼 스튜디오에서 제공하는 `매크로`를 이용할 수 있다.

`$(ProjectDir)` 매크로는 현재 프로젝트의 절대경로를 의미한다.

추가 포함 디렉터리는 추가한 폴더의 하위 폴더까지 전부 인식된다.

***

## 추가 라이브러리 디렉터리

헤더 파일을 포함시켰다면 실제 라이브러리를 링크시켜야한다.

설정 탭의 `링커`탭에서 **추가 라이브러리 디렉터리**를 수정하자.

[![img][additional-lib-dir]][additional-lib-dir]{: data-lightbox="falcon9-large" data-title="additional-lib-dir" }

실제 `.lib`파일이 위치한 경로를 이곳에 추가한다.

실제 컴파일 당시 이곳에서 추가한 라이브러리 파일이 링크된다.

### 추가 종속성

특정 라이브러리를 명시에서 입력해야 하는 경우도 존재하는데, 이런 경우 `링커 > 입력`탭의 `추가 종속성` 입력란을 이용한다.

[![img][additional-dep-lib]][additional-dep-lib]{: data-lightbox="falcon9-large" data-title="additional-dep-lib" }

라이브러리 파일 명을 앞서 했던 것처럼 똑같이 `<편집...>`란에서 추가한다.

***

추가 포함 디렉터리, 추가 라이브러리 디렉터리 설정을 마쳤다면 외부 라이브러리를 `#include`하고 빌드해서 제대로 작동하는지 확인하자.

[![img][lib-bulid]][lib-bulid]{: data-lightbox="falcon9-large" data-title="lib-bulid" }

[ext-lib-boost]: /assets/visualStudio/external-lib-boost.PNG
[proj-config]: /assets/visualStudio/sln-config.png
[additional-include-dir]: /assets/visualStudio/additional-include-dir.png
[additional-lib-dir]: /assets/visualStudio/additional-lib-dir.png
[additional-dep-lib]: /assets/visualStudio/additional-dependancy-lib.png
[lib-bulid]: /assets/visualStudio/lib-build.PNG
