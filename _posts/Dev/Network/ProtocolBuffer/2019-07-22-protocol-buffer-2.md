---
layout: post
title:  "Google protocol buffer (2) with C++"
date:   2019-07-22 18:20:00 +0900
author: Gnues
categories: network
tags:	network protocol-buffer google cpp
---

본 게시글은 [Protocol Buffer Basics: C++](https://developers.google.com/protocol-buffers/docs/cpptutorial) 글을 토대로 작성하였습니다.

***

## Why Use Protocol Buffers

구조화된 데이터를 어떻게 직렬화하고 어떻게 역직렬화를 해야 할까?

- 바이너리 형태의 raw 데이터
  - 이 데이터 포맷은 받거나 읽는 코드가 정확히 동일해야 한다.
  - 동일한 메모리 레이아웃으로 컴파일 되어야 하기 때문이다.
  - 그렇기 때문에 깨지기 쉬운(`breakable`) 형태가 된다.
  - 또한, 이 포맷으로 축적된 파일들과 연결된 어플리케이션들이 퍼지면 확장이 어렵다.

- 애드훅 형태 (단일 스트링 인코딩)
  - ex) `12:3-23:67` 처럼 4개의 정수로 인코딩된 형태
  - 간단하고 확장이 용이
  - 인코딩과 파싱하는 코드를 작성해야 함
  - 파싱은 약간의 런타임 비용을 부과
  - 매우 간단한 인코딩을 하는데에는 이 방법이 최선

- XML
  - XML은 가독성이 높고 많은 언어들로 바인딩된 라이브러리들이 존재
  - 다른 어플리케이션, 프로젝트와 데이터를 공유할 때 좋은 방안
  - 하지만 XML은 공간 집약적이고많은 퍼포먼스 패널티를 가져오는 인코딩, 디코딩으로 소문이 자자함
  - XML DOM 트리를 탐색하는 것은 클래스에서 필드를 탐색하는 것 보다 복잡함

프로토콜 버퍼는 이러한 문제점들을 해결할 수 있는 `flexible`, `efficient`, `automated` 특성을 가진 자동화 솔루션이다.

프토토콜 버퍼로 저장하려고 하는 데이터 구조에 대한 `.proto`파일을 작성한다.

그 파일로부터, 프로토콜 버퍼 컴파일러는 효율적인 바이너리 포맷의 프로토콜 버퍼 데이터를 자동으로 인코딩하고 파싱하는 것을 구현하는 클래스를 생성한다.

생성된 클래슨느 프로토콜 버퍼로 만들어진 필드들에 대한 `getter`, `setter`를 제공하고 하나의 유닛으로서 프로토콜 버퍼를 읽거나 쓰는 디테일한 부분을 책임진다.

또한, 프로토콜 버퍼 포맷은 이전 버전 포맷으로 인코딩된 데이터를 여전히 읽을 수 있게 하면서 포맷을 확장할 수 있다.

***

## Defining Your Protocol Format

예제에서 사용할 데이터는 사람들의 주소 정보를 파일로 쓰거나 읽어오는 매우 간단한 address book 어플리케이션이다.

주소록에 있는 각각의 사람들은 이름, ID, 이메일 주소, 전화번호를 가지고 있다.

가장 먼저 `.proto`파일부터 시작해야 한다. `.proto`파일에 있는 정의들은 간단하다.

직렬화하려는 각각의 데이터 구조들에 대한 메시지를 추가하고, 메시지에서 각각의 필드들에 대한 이름과 타입을 지정한다.

- `addressbook.proto`

```proto
package tutorial;

message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phone = 4;
}

message AddressBook {
  repeated Person person = 1;
}
```

- `package`

  - `.proto`파일은 `package`선언으로 시작한다.

  - 패키지 선언은 서로 다른 프로젝트들 간의 이름 혼선을 방지하기 위함이다.

  - `C++`에서는 생성된 클래스들은 패키지 이름과 동일한 `namespace`안에 위치할 것이다.

- `message`

  - 패키지 선언 후에 자신이 정의할 데이터 구조에 맞는 메시지 정의들이 나온다.

  - 메시지는 타입 형태의 필드들을 포함하는 집합체다.

  - 간단한 표준 데이터 타입들이 필드 타입으로 사용된다.
    - `bool`, `int32`, `float`, `double`, `string` 등

  - 필드 타입으로 그 외의 메시지 타입들을 이용할 수도 있다.
    - ex) `Person`메시지가 `PhoneNumber`메시지를 포함하고 있다.
    - ex) `AddressBook`aptlwlsms `Person`메시지들을 포함하고 있다.

  - 다른 메시지 안쪽에서 메시지 타입을 정의할 수도 있다.

  - 필드들에 `enum`같은 상수를 정의하여 사용할 수도 있다.

- `field`

  - 각각의 요소들의 `= 1`, `= 2` 마커들은 필드가 바이너리 인코딩에서 사용하는 유니크한 태그를 식별하도록 해준다.

  - 태그 넘버 `1 ~ 15`는 인코딩 하는데 `1` 이하의 바이트를 요구한다.

  - 따라서, 일반적으로 사용되거나 반복적인 요소들에 대해서 좋은 최적화 방안이다.

  - 각각의 필드는 다음의 `modifier` 중에 하나를 포함해야 한다.
    - **주의** : `proto3`버전에서는 `required`, `optional`, `default` 지시자가 더 이상 사용되지 않는다.

    - `required` : 이 필드의 값은 반드시 제공되어야 함
      - 제공되지 않으면 초기화되지 않은 것으로 간주될 것이다.
      - `libprotobuf`가 디버그 모드에서 컴파일되면, 초기화되지 않은 메시지의 직렬화는 실패하게 된다.
      - 최적화 빌드에서는 이러한 체크가 스킵되고 메시지는 그냥 작성될 것이다.
      - 하지만 초기화되지 않은 메시지를 파싱하는 것은 항상 실패할 것이다.

    - `optional` : 이 필드는 셋팅되도 되고 안되도 된다.
      - 필드가 셋팅되지 않으면 기본값이 사용된다.
      - 숫자에서는 `0`, 스트링에서는 빈 스트링, `bool`에서는 `false`가 사용된다.
      - 임베디드된 메시지들에 대해서는 기본 값은 항상 필드에 셋팅된 값이 없다는 메시지의 `default instance` 혹은 `prototype`이다.
      - 값이 셋팅되지 않은 필드의 값을 가져올 때 항상 그 필드의 기본값을 리턴한다.

    - `repeated` : 이 필드는 `0`번 이상 반복된다.
      - 반복되는 값들의 순서는 프로토콜 버퍼에 보존된다.
      - 이 필드는 `dynamic`하게 사이즈가 변하는 배열로 생각할 수 있다.

### 주의

> `Required is Forever`
>
> 특정 시점에서 `required` 필드를 쓰거나 전송하는 것을 멈추려고 `optional`로 바꾸면 문제가 생길수도 있다.
>
> 이전 `reader`들은 이 필드가 없는 메시지들을 미완성 메시지로 간주하고 모두 드랍시키거나 거부할 수도 있다.

***

## References

- <https://developers.google.com/protocol-buffers/docs/overview>
