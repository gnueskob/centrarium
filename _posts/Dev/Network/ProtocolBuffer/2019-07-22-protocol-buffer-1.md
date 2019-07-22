---
layout: post
title:  "Google protocol buffer (1)"
date:   2019-07-22 18:20:00 +0900
author: Gnues
categories: network
tags:	network protocol-buffer google
---

## What are protocol buffers

구글은 프로토콜 버퍼를 이렇게 소개한다.

- language-neutral
- platform-neutral
- extensible
- way of serializing structured data for use in communications protocols, data storage, and more

언어 중립적, 플랫폼 중립적, 확장성을 지닌 직렬화 매커니즘이자 데이터 포맷이다.

XML보다 작고, 빠르고 간단하며 데이터 포맷을 정의하기만 하면 컴파일을 통해 여러 언어형태의 코드로 바꿀 수 있다.

통신이나 데이터 저장에 있어서 데이터를 직렬화(`Serialize`)/역직렬화(`Deserialize`)하던 과정을 직접해야 했지만 프로토콜 버퍼를 이용하면 훨씬 쉽게 다룰 수 있다.

심지어 이미 배포되어서 이전 데이터 구조를 다루는 프로그램에 상관없이 데이터 구조를 수정할 수 있다.

***

## How do they work

직렬화하려는 데이터 구조는 `.proto` 파일 포맷으로 정의해야 한다.

각 프로토콜 버퍼의 메시지는 여러개의 `name-value` 쌍을 포함하는 작은 논리적 정보이다.

- `.proto` 파일 예제

```proto
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
```

위와 같이 메시지 포맷은 굉장히 단순하다.

각 메시지 타입은 한 개 이상의 고유 번호 필드를 가지며 각 필드들은 `name`, `value type`을 가진다.

`value type`은 `numbers`, `boolean`, `strings`, `raw bytes` 등의 타입이 될 수 있으며 **데이터 계층구조**도 만들 수 있다.

각 필드는 `optional`, `required`, `repeated`의 유형을 정할 수 있다.

`.proto`파일에 메시지(데이터 구조)를 정의하고 나면, 프로토콜 버퍼 컴파일러를 통해 컴파일하여 자신이 원하는 언어로 `data access classes`을 생성할 수 있다.

이 클래스들은 간단한 접근자(`accessor`)와 직렬화/역직렬화(파서) 메소드를 제공한다.

`.proto`파일을 `C++` 언어기준으로 컴파일하여 생성한다면 아래처럼 사용할 수 있는 클래스를 생성해준다.

- 자신이 만든 데이터 구조를 갖는 클래스에 값을 설정하고 파일로 저장

```cpp
Person person;
person.set_name("John Doe");
person.set_id(1234);
person.set_email("jdoe@example.com");
fstream output("myfile", ios::out | ios::binary);
person.SerializeToOstream(&output);
```

- 저장한 프로토콜 버퍼 메시지 파일을 읽기

```cpp
fstream input("myfile", ios::in | ios::binary);
Person person;
person.ParseFromIstream(&input);
cout << "Name: " << person.name() << endl;
cout << "E-mail: " << person.email() << endl;
```

만약 메시지 포맷에 에 새로운 필드를 추가할 경우 오래된 바이너리는 파싱할 때 새로운 필드를 단순히 무시함으로써 `backward-compatibility`를 깨지 않고 작동한다.

## Why not just use XML

프로토콜 버퍼는 XML보다 많은 이점이 있다.

- simpler
- 3 ~ 10 times smaller
- 20 ~ 100 times faster
- less ambiguous
- generate data access classes that are easier to use programmatically

`name`, `email`을 갖는 `person` 모델을 예로 들어보자.

- `XML`의 경우

```xml
<person>
  <name>John Doe</name>
  <email>jdoe@example.com</email>
</person>
```

- `Protcol buffer`의 경우

```proto
# Textual representation of a protocol buffer.
# This is *not* the binary format used on the wire.
person {
  name: "John Doe"
  email: "jdoe@example.com"
}
```

이 메시지가 `protocol buffer binary format`으로 인코딩 될 때, 약 28 bytes 크기를 갖고 약 100 ~ 200 nanosec 시간만이 걸릴 뿐이다.

`XML` 버전의 경우엔 최소 69 bytes(공백을 제거할 시)와 5,000 ~ 10,000 nanosec의 시간이 걸린다.

데이터를 다루는 것 또한 xml에 비해 매우 쉽다.

```cpp
// protocol buffer를 사용한 person class
cout << "Name: " << person.name() << endl;
cout << "E-mail: " << person.email() << endl;
```

```cpp
// xml을 parsing 하는 경우
cout << "Name: "
     << person.getElementsByTagName("name")->item(0)->innerText()
     << endl;
cout << "E-mail: "
     << person.getElementsByTagName("email")->item(0)->innerText()
     << endl;
```

## 사용방법

[Download the package](https://developers.google.com/protocol-buffers/docs/downloads) 링크에서 최신 버전을 받을 수 있다.

해당 패키지는 Java, Python, C++ 언어의 protocol buffer compiler를 포함한다.

Build, Install 방법은 `README`에 기재된 순서대로 따라하면 되며 각 언어에 맞게 [튜토리얼](https://developers.google.com/protocol-buffers/docs/tutorials)을 제공한다.

1. 데이터 구조를 `.proto` 파일로 작성
2. Protocol Buffer Complier인 `protoc`를 실행하여 `.proto` 파일을 자신이 원하는 특정 언어 코드로 컴파일
3. 컴파일된 언어 코드를 `include` 혹은 `import`
4. 컴파일된 코드에서 API를 가져다 사용

## References

- <https://developers.google.com/protocol-buffers/docs/overview>
