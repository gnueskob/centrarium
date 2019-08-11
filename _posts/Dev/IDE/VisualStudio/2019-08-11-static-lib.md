---
layout: post
title:  "Visual Studio 2019 - static lib project"
date:   2019-08-11 22:10:00 +0900
author: Gnues
categories: editor
tags:	editor visual-studio static-library
---

이 글에선 Visual Studio에서 정적 라이브러리 프로젝트 제작 방법을 소개한다.

Visual Studio 2019를 기준으로 작성되었다.

***

## 프로젝트 생성

비주얼 스튜디오를 통해 새 프로젝트를 생성할 때 **정적 라이브러리** 항목을 선택하여 생성한다.

[![img][staticLibProject]][staticLibProject]{: data-lightbox="falcon9-large" data-title="staticLibProject" }

혹은 이미 생성된 프로젝트의 설정 창에서 **구성 형식**을 **정적 라이브러리**로 바꾼다.

[![img][staticLibProject2]][staticLibProject2]{: data-lightbox="falcon9-large" data-title="staticLibProject2" }

***

## 정적 라이브러리 프로젝트 참조

현재 작업중인 솔루션에 정적 라이브러리 프로젝트를 만들고, 해당 라이브러리를 사용할 별도의 프로젝트를 생성한다.

[![img][staticLibSln]{: style="max-width: 55%; height: auto;" }][staticLibSln]{: data-lightbox="falcon9-large" data-title="staticLibSln" }

위 이미지 속 솔루션에는 정적 라이브러리 프로젝트(StaticLib)와 일반 애플리케이션 프로젝트(Chapter1)가 존재한다.

애플리케이션 프로젝트의 소스파일에서 정적 라이브러리를 `#include`하여 사용할 것이다.

그러기 위해선 애플리케이션 프로젝트에서 정적 라이브러리 프로젝트를 **참조**해야 한다.

솔루션 탐색기의 애플리케이션 프로젝트에서 우클릭 후 `추가 > 참조`를 클릭하거나 해당 프로젝트 `참조`에서 우클릭 후 `참조 추가`를 클릭한다.

[![img][staticLibRef]{: style="max-width: 80%; height: auto;" }][staticLibRef]{: data-lightbox="falcon9-large" data-title="staticLibRef" }

[![img][staticLibRef2]{: style="max-width: 60%; height: auto;" }][staticLibRef2]{: data-lightbox="falcon9-large" data-title="staticLibRef2" }

참조 추가 창이 활성화 되면, 기존에 생성해두었던 정적 라이브러리 프로젝트가 참조 가능한 목록에 존재할 것이다.

체크 후 확인 버튼을 눌러 애플리케이션에서 정적 라이브러리 프로젝트를 참조하도록 설정한다.

[![img][staticLibRefAdd]][staticLibRefAdd]{: data-lightbox="falcon9-large" data-title="staticLibRefAdd" }

제대로 참조 추가를 완료했다면 아래 이미지 처럼 애플리케이션 프로젝트 하단 참조 탭에 정적 라이브러리 프로젝트가 추가될 것이다.

[![img][staticLibRefResult]{: style="max-width: 60%; height: auto;" }][staticLibRefResult]{: data-lightbox="falcon9-large" data-title="staticLibRefResult" }

***

## 라이브러리 작성

이제 라이브러리에서 제공할 샘플 클래스를 작성해보자.

```cpp
// sampleLib.h
#pragma once

namespace sampleStaticLib
{
  class SampleLib
  {
  public:
    void func();
  };
}
```

```cpp
// sampleLib.cpp
#include <iostream>
#include "sampleLib.h"

namespace sampleStaticLib
{
  void SampleLib::func()
  {
    std::cout << "Hello, world!" << std::endl;
  }
}
```

위 라이브러리에서 제공하는 `sampleLib` 클래스는 `"Hello, world!"`를 출력하는 단순한 함수만을 가지고 있다.

또한, `sampleStaticLib`라는 네임스페이스에 존재한다.

라이브러리에서 제공하는 이 클래스를 애플리케이션에서 사용할 수 있다.

애플리케이션 프로젝트에서 라이브러리를 `include`하기 전에 현재 솔루션의 폴더 구조를 한 번 살펴보자.

```text
├─Chapter1 (application project)
│  ├─lec01.cpp
│  └─...
│
├─StaticLib (static library project)
│  ├─sampleLib.h
│  ├─sampleLib.cpp
│  └─...
│
└─...
```

각 프로젝트는 별도의 폴더로 구현되어있으며 각 폴더는 같은 레벨(상위 폴더가 같음)이다.

때문에 애플리케이션의 소스코드에서 우리가 생성한 라이브러리를 `include`하려면 다음 처럼 헤더를 포함시킨다.

```cpp
#include <iostream>
#include "../StaticLib/sampleLib.h"

int main()
{
  sampleStaticLib::SampleLib sampleLibObj;
  sampleLibObj.func();

  return 0;
}
```

사용자의 환경에 따라 얼마든지 변할 수 있으므로 각자 알맞게 설정하면 된다.

***

## 주의 및 확인

라이브러리에 따라 프로젝트 플랫폼 설정, `C/C++ > 코드생성`의 런타임 라이브러리 설정을 맞춰야 한다.

빌드해서 제대로 작동하는지 확인하자.

[![img][staticLibBuild]{: style="max-width: 80%; height: auto;" }][staticLibBuild]{: data-lightbox="falcon9-large" data-title="staticLibBuild" }

이처럼 같은 솔루션에 정적 라이브러리를 직접 생성하고 애플리케이션에서 참조하여 사용할 수 있다.

혹은, 정적 라이브러리만을 따로 빌드하여 [이곳][externalLibPost]에서 설명하는 대로 **헤더**와 **.lib**파일을 포함시킬 수도 있다.

[staticLibProject]: /assets/visualStudio/staticLibProject.png
[staticLibProject2]: /assets/visualStudio/staticLibProject2.png
[staticLibSln]: /assets/visualStudio/staticLibSln.png
[staticLibRef]: /assets/visualStudio/staticLibRef.png
[staticLibRef2]: /assets/visualStudio/staticLibRef2.png
[staticLibRefAdd]: /assets/visualStudio/staticLibRefAdd.png
[staticLibRefResult]: /assets/visualStudio/staticLibRefResult.png
[staticLibBuild]: /assets/visualStudio/staticLibBuild.png

[externalLibPost]: https://gnueskob.github.io/editor/2019/08/11/external-lib.html
