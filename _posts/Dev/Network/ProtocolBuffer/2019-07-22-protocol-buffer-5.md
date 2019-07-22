---
layout: post
title:  "Google protocol buffer (5) with C++"
date:   2019-07-22 18:20:00 +0900
author: Gnues
categories: network
tags:	network protocol-buffer google cpp
---

본 게시글은 [Protocol Buffer Basics: C++](https://developers.google.com/protocol-buffers/docs/cpptutorial) 글을 토대로 작성하였습니다.

***

## Extending a Protocol Buffer

프로토콜 버퍼를 사용하는 코드를 배포한 뒤 프로토콜 버퍼 정의를 확장이 필요할 수 있다.

하위 호환이 가능한 새로운 버퍼는 다음 규칙들을 따라야 한다.

- 존재하고 있는 어떤 필드라도 태그 넘버를 변경해서는 안 된다.
- required 필드들은 추가하거나 제거해서는 안 된다.
- optional이나 repeated 필드들은 제거해도 괜찮다.
- optional이나 repeated 필드들은 추가되어도 괜찮지만, 반드시 새로운 태그 넘버들을 사용해야 한다
  - 즉, 이 프로토콜 버퍼에서 절대 사용되지 않았던 태그 넘버 - 지워진 필드의 것도 안 된다

규칙을 따르면 이전 코드는 새롭게 확장된 메시지를 읽을 수 있고, 새롭게 추가된 필드들은 무시할 수 있다.

***

## Optimization Tips

`C++` 프로토콜 버퍼 라이브러리는 최적화가 잘 되어있다.

하지만 알맞은 사용법이 좀 더 퍼포먼스를 향상 시킬 수 있다.

- 가능하다면 메시지 오브젝트들을 재사용하자.
  - 메시지들을 삭제되더라도 재사용에 할당된 메모리 주변에 있으려 할 것.
  - 연속으로 동일한 타입과 유사한 구조를 사용하는 많은 메시지들을 다루면 메모리 할당자의 부담을 줄여주기 위해서 같은 오브젝트를 사용하는 것이 좋음
  - 메시지들이 모양을 바꾸거나 가끔 일반적인 경우보다 큰 메시지를 만들면 오랜 시간동안 오브젝트가 확장될 수 있음
  - [SpaceUsed](https://developers.google.com/protocol-buffers/docs/reference/cpp/google.protobuf.message#Message.SpaceUsed.details)를 사용해서 메시지 오브젝트들의 크기를 감시하고 너무 커졌을 경우에는 지워주는 것이 좋음

- 시스템의 메모리 할당자는 여러 쓰레드로부터 많은 수의 작은 크기의 오브젝트들을 할당하는 것에 대해서 최적화가 잘 되어 있지 않을 것이다.
  - 대신에 [Google's tcmalloc](https://github.com/gperftools/gperftools)를 사용해 보자.

***

대게 프로젝트에서는 직접 이용하기 보다는 별도의 `Converter`를 만들어두고, **통신시 혹은 직렬화 시에만 Convert**해서 사용한다.

`Protobuf` 통신은 `application/x-google-protobuf`와 같은 새로운 형식이며 서버 - 클라이언트간 통신 시 이 형태로 `consume`, `produce`가 일어나야 한다.

이때 서버와 클라이언트는 동일한 `.proto`파일을 공유하기만 하면 각기 환경에 알맞게 컴파일해서 내부 클래스로 사용하면 된다.

***

## References

- <https://developers.google.com/protocol-buffers/docs/overview>
