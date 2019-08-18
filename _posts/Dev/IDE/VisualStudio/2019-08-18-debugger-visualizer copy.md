---
layout: post
title:  "Visual Studio 2017 - C++ Debugger Visualizers"
date:   2019-08-18 19:00:00 +0900
author: Gnues
categories: editor
tags:	editor visual-studio debug debugger-visualizer
---

이번 글에서는 **C++ Debugger Visualizers**에 대해 소개한다.

VS2017 버전까지 지원하며 Boost, wxWidgets, TinyXML, TinyXML2 라이브러리를 지원한다.

아직까지는 VS2019 버전은 지원하지 않는다.

[Marketplace Link][marketPlace]

[Github Link][github]

C++ Debugger Visualizers를 소개하기전에 먼저 Visualizers가 무엇인지 소개하고 넘어가겠다.

***

## Visualizers

비주얼 스튜디오에서는 디버깅 중 개체위에 커서를 올리면 조사식처럼 개체 정보를 나타내주는 창(**DataTip**)을 팝업시킨다.

[![img][dataTip]][dataTip]{: data-lightbox="falcon9-large" data-title="dataTip" }

혹은 조사식에서도 동일하게 개체의 정보를 확인할 수 있다.

[![img][watch]][watch]{: data-lightbox="falcon9-large" data-title="watch" }

개체 정보중에 특정 타입의 경우에는 돋보기가 나타나며 클릭해보면 여러가지 **시각화 도우미** 들이 목록에 나타날 것이다.

[![img][visualizers]][visualizers]{: data-lightbox="falcon9-large" data-title="visualizers" }

이것이 Visualizer 이다.

내장되어 있는 Visualizer는 총 4가지이며 이 Type의 개체를 디버깅 할때만 Visualizer가 발동된다.

- Text
- Html
- Xml
- DataSet

VS2005 버전 이후로는 내장된 Visualizer 이외에도 커스터마이징하여 자신만의 Visualizer를 제작할 수 있다.

예로, 이미지개체를 처리할 때 실제로 이미지를 볼 수 있도록 사용할 수도 있다.

[![img][ImgVisualizer]][ImgVisualizer]{: data-lightbox="falcon9-large" data-title="ImgVisualizer" }

비주얼 스튜디오는 이렇게 디버깅 시 개발자가 빈번하게 접하는 데이터 시각화를 커스터마이징하는 기능을 제공한다.

이를 통해 사용자 정의 시각화 모듈을 생성해 디버깅에 유용하게 활용할 수 있다.

다음의 이미지를 통해 쉽게 이해할 수 있을 것이다.

[![img][visualizerBefore]][visualizerBefore]{: data-lightbox="falcon9-large" data-title="visualizerBefore" }

> 커스텀 데이터 시각화 적용 전

[![img][visualizerAfter]][visualizerAfter]{: data-lightbox="falcon9-large" data-title="visualizerAfter" }

> 커스텀 데이터 시각화 적용 후

***

## Natvis Visualizer

비주얼 스튜디오 디버깅 창의 '텍스트 시각화 사용자 정의 규칙'은 **autoexp.dat** 파일을 통해 정의되는데, 이러한 문법을 통해 **자동, 지역식, 조사식** 창에서 변수값을 조회할 수 있다.

오랜 기간 비주얼 스튜디오를 사용했다면 비주얼 C++의 STL (Stan dard Template Library)로 고생했던 경험이 한 번쯤은 있을 것이다.

예전에는 STL의 난해한 에러 코드와 경고 메시지, 텍스트 시각화로 인해 인스턴스를 포함한 멤버 변수를 찾는 데에만 상당한 시간이 걸렸기 때문이다.

이러한 불편 때문인지 비주얼 스튜디오는 새로운 버전이 출시될 때마다 텍스트 시각화 기능이 점차 개선돼왔다.

텍스트 시각화가 보기 편해졌을 뿐 아니라 디버깅 시에도 원하는 정보를 보다 쉽게 찾을 수 있게 됐다.

이러한 변화의 중심에는 텍스트 사용자 정의 규칙의 업데이트가 있었다.

VS 2012에서는 텍스트 시각화 사용자 정의 규칙이 대대적으로 개선됐다.

기존 autoexp.dat를 이용한 정의 방식의 문제점을 보완하고, 개발자가 사용하기 쉽도록 xml 문법으로 규칙을 기술하도록 변경됐다.

또한 텍스트 시각화 사용자 정의 규칙을 정의할 때 발생할 수 있는 문제점에 대한 추적도 한결 쉬워졌다.

게다가 정의 파일에 대한 버전관리나 여러 파일로 나누어 프레임워크별, 규칙별, 프로젝트별로 규칙을 적용할 수 있는 **Natvis** 프레임워크도 추가됐다.

> Nativis 프레임워크를 통해 visualizer를 만드는 방법은 아래 링크를 참조
>
> - [Create custom views of C++ objects in the debugger][msNatvis]
> - [네이티브 환경에서 사용자 정의 텍스트 시각화 만들기 (1)][Kdata1]
> - [네이티브 환경에서 사용자 정의 텍스트 시각화 만들기 (2)][Kdata2]
> - [네이티브 환경에서 사용자 정의 텍스트 시각화 만들기 외전][Kdata3]

***

## C++ Debugger Visualizers

**C++ Debugger Visualizers**는 여러가지 라이브러리를 위해 설계된 Visualizer이다.

문서의 머릿말에 소개한 것 처럼 VS2017 버전까지 지원하며 Boost, wxWidgets, TinyXML, TinyXML2 라이브러리를 지원한다.

하지만 아직까지는 VS2019 버전은 지원하지 않는다.

VS2017 버전에서는 [Marketplace][marketPlace]에서 쉽게 설치할 수 있다.

[![img][debuggerVisualizer]][debuggerVisualizer]{: data-lightbox="falcon9-large" data-title="debuggerVisualizer" }

해당 visualizer를 설치하고 지원하는 라이브러리를 사용해보면 디버깅을 보다 편히할 수 있을 것이다.

아래 이미지는 boost 라이브러리의 multi-index를 사용하여 visualizer 유무 차이를 보여준다.

[![img][multiIndex2]][multiIndex2]{: data-lightbox="falcon9-large" data-title="multiIndex2" }

> boost library - multi-index 디버깅 테스트 (C++ Debugger Visualizers 적용 X)

[![img][multiIndex]][multiIndex]{: data-lightbox="falcon9-large" data-title="multiIndex" }

> boost library - multi-index 디버깅 테스트 (C++ Debugger Visualizers 적용)

이처럼 visualizer를 통해 자신만의 디버깅 환경을 구축하여 보다 쉽고 빠르게 디버깅을 할 수 있다.

***

## References

- [방실 블로그][bangsil]
- [한국 데이터 산업 진흥원][Kdata1]
- [MicroSoft VS docs][msNatvis]
- [C++ Debugger Visualizers][github]

[dataTip]: /assets/visualStudio/dataTip.png
[watch]: /assets/visualStudio/watch.png
[visualizers]: /assets/visualStudio/visualizers.png
[ImgVisualizer]: /assets/visualStudio/imageVisualizer.jpg
[visualizerBefore]: /assets/visualStudio/visualizer_before.png
[visualizerAfter]: /assets/visualStudio/visualizer_after.png
[debuggerVisualizer]: /assets/visualStudio/debuggerVisualizer.png
[multiIndex]: /assets/visualStudio/testMultiIndexVisualizer.png
[multiIndex2]: /assets/visualStudio/testMultiIndexVisualizer2.png

[bangsil]: http://www.bangsil.net/MyPosts/?postId=13da98de-2d3f-4d03-80ba-8e422407f15b
[Kdata1]: https://www.kdata.or.kr/info/info_04_view.html?field=title&keyword=%BB%E7%BF%EB%C0%DA%20%C1%A4%C0%C7%20%C5%D8%BD%BA%C6%AE%20%BD%C3%B0%A2%C8%AD&type=techreport&page=1&dbnum=177748&mode=detail&type=techreport
[Kdata2]: https://www.kdata.or.kr/info/info_04_view.html?field=title&keyword=%BB%E7%BF%EB%C0%DA%20%C1%A4%C0%C7%20%C5%D8%BD%BA%C6%AE%20%BD%C3%B0%A2%C8%AD&type=techreport&page=1&dbnum=178647&mode=detail&type=techreport
[Kdata3]: https://www.kdata.or.kr/info/info_04_view.html?field=title&keyword=%BB%E7%BF%EB%C0%DA%20%C1%A4%C0%C7%20%C5%D8%BD%BA%C6%AE%20%BD%C3%B0%A2%C8%AD&type=techreport&page=1&dbnum=179747&mode=detail&type=techreport
[msNatvis]: https://docs.microsoft.com/en-us/visualstudio/debugger/create-custom-views-of-native-objects?view=vs-2019

[marketPlace]: https://marketplace.visualstudio.com/items?itemName=ArkadyShapkin.CDebuggerVisualizersforVS2017
[github]: https://github.com/KindDragon/CPPDebuggerVisualizers/issues
