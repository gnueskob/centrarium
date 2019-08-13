---
layout: post
title:  "Google protocol buffer (6) with C++"
date:   2019-08-14 00:40:00 +0900
author: Gnues
categories: network
tags:	network protocol-buffer google cpp
---

이전까지 프로토콜 버퍼(이하 protobuf) C++버전의 튜토리얼 내용을 따라해보았다.

이번 포스팅에서는 protobuf를 빌드하는 방법을 알아본다.

***

## Download

이전 포스팅에서 소개한 바와 같이 [이곳](https://github.com/protocolbuffers/protobuf/releases)에서 최신버전의 릴리즈를 받을 수 있다.

windows, MSVC++ 환경으로 작업하기 때문에 **protobuf-cpp-__version__.zip**를 받고 압축을 풀어준다.

**src** 폴더에는 실제 프로젝트에서 포함시켜 사용하는 헤더파일들이 존재한다.

**cmake** 폴더에는 cmake 빌드에 필요한 파일들이 존재한다.

***

## Cmake

protobuf를 빌드하기 위해 **Cmake**가 필요하다.

Cmake를 사용하는 것에는 여러가지 방법이 존재하고, windows/MSVC 환경에서는 **Cmake-gui**를 이용할 수 있다.

[Cmake-gui](https://cmake.org/download/)를 받고 실행시키면 아래와 같은 창이 뜰 것이다.

[![img][cmake]{: style="max-width: 70%; height: auto;" }][cmake]{: data-lightbox="falcon9-large" data-title="cmake" }

빨간 박스의 경로는 다운받은 protobuf 파일중 **'CMakeLists.txt'**파일이 위치하는 폴더로 지정한다. (보통은 **cmake**폴더에 존재할 것이다.)

파란 박스의 경로는 cmake 빌드결과물이 저장될 위치로 지정한다.

위 이미지에서는 다운받은 protobuf 폴더 내부에 새로운 폴더 **'cmake_build'**를 만들어 지정하였다.

경로 지정이 끝났으면 빌드할 컴파일러 toolset을 지정해줘야 한다.

초록 박스의 **'Configure'** 버튼을 클릭하여 설정할 수 있다.

[![img][cmakeConfig]{: style="max-width: 70%; height: auto;" }][cmakeConfig]{: data-lightbox="falcon9-large" data-title="cmakeConfig" }

설치된 비주얼 스튜디오 버전을 선택하여 Finish 버튼을 클릭하면 아래와 같이 **'Generate'** 준비가 끝나게 된다.

[![img][cmakeGenerate]{: style="max-width: 70%; height: auto;" }][cmakeGenerate]{: data-lightbox="falcon9-large" data-title="cmakeGenerate" }

**Generate** 버튼을 눌러 비주얼 스튜디오 솔루션을 생성시키자.

Generate가 끝나면 cmake-gui 제일 하단 로그 패널에 `Generating done` 문구가 뜨고 초기에 설정했던 빌드 결과물 경로(ex. cmake_build)에 여러 VS 프로젝트들과 솔루션(protobuf.sln)이 생성될 것이다.

***

## Visual Studio Sln Build

해당 솔루션을 실행시켜서 확인해보면 **libprotobuf** 프로젝트가 존재하는데, 이 프로젝트를 빌드시켜 정적 라이브러리 파일(`.lib` )을 얻을 수 있다.

솔루션 구성에 따라 디버깅(`libprotobufd.lib`), 릴리즈(`libprotobuf.lib`) 모드별 라이브러리를 생성할 수 있다.

[![img][libBuild]{: style="max-width: 50%; height: auto;" }][libBuild]{: data-lightbox="falcon9-large" data-title="libBuild" }

**'protoc'** 프로젝트는 protobuf 컴파일러 프로젝트이다. 해당 프로젝트를 빌드하면 **'protoc.exe'** 실행파일을 얻을 수 있으며 이것으로 `.proto`파일을 컴파일하여 cpp 소스, 헤더파일로 변환할 수 있다.

또한, protoc 프로젝트는 libprotobuf 프로젝트를 참조하고 있어서 protoc 프로젝트만 빌드해도 `.lib`은 함께 생성된다.

글 앞부분의 [다운로드 링크](https://github.com/protocolbuffers/protobuf/releases)에서 이미 빌드된 'protoc.exe' 파일을 별도로 받을 수도 있다. (**protoc-3.9.1-winXX.zip**)

> **'ALL_BUILD'** 프로젝트를 빌드하면 해당 솔루션 내에 존재하는 모든 프로젝트가 같이 빌드된다. 이 프로젝트를 빌드해도 상관없다.

***

## #include to Project

이제 protobuf를 사용할 프로젝트에서 라이브러리를 포함시켜보자.

`C/C++ > 추가 포함디렉터리`에 **src**폴더를 추가하고, `링커 > 추가 종속성`에 빌드했던 라이브러리를 추가하자.

이제 프로젝트 내에서 protobuf 라이브러리를 사용할 수 있다.

```cpp
...
#include <google/protobuf/message.h>
...

...
bool ParseDataToProto(google::protobuf::Message* pProto, char* pData, short size);
...
```

VS2019에서 외부 라이브러리를 사용하는 것은 [이곳][externalLibPost]을 참조하자.

[cmake]: /assets/protobuf/cmake.png
[cmakeConfig]: /assets/protobuf/cmakeConfig.png
[cmakeGenerate]: /assets/protobuf/cmakeGenerate.png
[libBuild]: /assets/protobuf/libBuild.png
[externalLibPost]: https://gnueskob.github.io/editor/2019/08/11/external-lib.html
