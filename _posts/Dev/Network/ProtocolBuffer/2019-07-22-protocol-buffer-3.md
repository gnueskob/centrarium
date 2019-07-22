---
layout: post
title:  "Google protocol buffer (3) with C++"
date:   2019-07-22 18:20:00 +0900
author: Gnues
categories: network
tags:	network protocol-buffer google c++
---

본 게시글은 [Protocol Buffer Basics: C++](https://developers.google.com/protocol-buffers/docs/cpptutorial) 글을 토대로 작성하였습니다.

***

## Compiling Your Protocol Buffers

`.proto`파일을 생성했으니, 이제 `AddressBook`메시지를 쓰고 읽을 수 있는 클래스를 생성해야 한다.

프로토콜 버퍼 컴파일러 `protoc`를 자신의 `.proto`파일에 실행하자.

```sh
protoc -l=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/addressbook.proto
```

- `-l` : 자신의 어플리케이션 소스가 있는 곳 (default: 현재 디렉토리가 사용됨)
- `--cpp_out` : 클래스 코드를 생성하려는 디렉토리
- `.proto`파일이 있는 경로

해당 명령을 통해 컴파일을 진행하면 `addressbook.ph.h`, `addressbook.pb.cc` 파일이 생성된다.

- `addressbook.ph.h` : 생성된 클래스들을 정의하는 헤더 파일
- `addressbook.pb.cc` : 클래스들의 구현 부분을 포함하는 파일

***

## The Protocol Buffer API

생성된 코드들은 다소 복잡하게 되어있다. `Person` 클래스를 좀더 들여다 보면 아래와 같은 함수들을 포함하고 있을 것이다.

```cpp
// name
inline bool has_name() const;
inline void clear_name();
inline const ::std::string& name() const;
inline void set_name(const ::std::string& value);
inline void set_name(const char* value);
inline ::std::string* mutable_name();

// id
inline bool has_id() const;
inline void clear_id();
inline int32_t id() const;
inline void set_id(int32_t value);

// email
inline bool has_email() const;
inline void clear_email();
inline const ::std::string& email() const;
inline void set_email(const ::std::string& value);
inline void set_email(const char* value);
inline ::std::string* mutable_email();

// phone
inline int phone_size() const;
inline void clear_phone();
inline const ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >& phone() const;
inline ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >* mutable_phone();
inline const ::tutorial::Person_PhoneNumber& phone(int index) const;
inline ::tutorial::Person_PhoneNumber* mutable_phone(int index);
inline ::tutorial::Person_PhoneNumber* add_phone();
```

- `commons`
  - `getter`들은 필드 이름과 동일하게 소문자로 구현되어 있고 `setter`들은 `set_`으로 시작한다.
  - 각각의 단일 필드(`required`, `optional`)에 대해서 값이 셋팅 되어있는지 여부를 알려주는 `has_`메소드들도 있다. (`proto2` 전용)
  - 또한, 각 필드의 값을 비우는 `clear_`메소드 들도 존재한다.

- `string extra method`
  - `name`, `email` 필드는 문자열 타입으로 몇 개의 추가 메소드들을 가지고 있다.
  - 스트링에 대한 포인터를 바로 가져오는 `mutable_getter`와 타입별 추가 `setter`이다.
  - 해당 스트링에 값이 아직 셋팅되지 않아도 `mutable_` 메소드는 호출할 수 있으며, 셋팅이 안된 스트링은 빈 스트링으로 초기화된다.

- `repeated extra method`
  - `repeated` 필드들은 몇 가지 특별한 메소드를 가지고 있다.
  - `_size` : 할당된 필드 요소들이 몇 개 인지
  - `index`를 통해 지정된 요소 얻기
  - 지정된 `index`의 요소를 업데이트
  - `add_` : 필드에 새로운 요소를 추가하고 편집

> 더 정확한 메소드들의 역할 : [C++ generated code reference](https://developers.google.com/protocol-buffers/docs/reference/cpp-generated)

***

## Enums and Nested Classes

- `Enum`
  - 생성된 코드에는 `.proto`파일에 있는 `enum`과 대응되는 `PhoneType enum`을 포함하고 있다.
  - `Person::PhoneType`에서 조회해 볼 수 있다.

- `Nested class`
  - 생성된 코드에는 `Person::PhoneNumber` 중첩 클래스도 존재한다.
  - 실제로는 `Person_PhoneNumber`로 별도의 클래스를 생성한다.
  - `Person`클래스 내부에서 `typedef`를 통해 중첩 클래스처럼 사용한다.

***

## Standard Message Methods

각각의 메시지 클래스들은 전체 메시지를 체크하고 조작할 수 있는 몇 가지 서로 다른 메소드들을 가지고 있다.

- `bool IsInitialized() const` : 모든 required 필드들이 셋팅되어 있는지 체크
- `string DebugString() const` : 읽을 수 있는 형태로 메시지를 리턴한다. 특히 디버깅 시에 유리하다.
- `void CopyFrom(const Person& from)` : 전달해 준 값으로 메시지를 덮어 쓴다.
- `void Clear()` : 모든 요소들을 빈 상태로 지운다.

> 해당 메소드들은 C++ 프로토콜 버퍼 클래스들에 의해서 공유되는 [Message 인터페이스 구현 섹션](https://developers.google.com/protocol-buffers/docs/reference/cpp/google.protobuf.message#Message)에서 설명한다.

***

## Parsing and Serialization

각각의 프로토콜 버퍼 클래스는 프로토콜 버퍼 [바이너리 포맷](https://developers.google.com/protocol-buffers/docs/encoding)을 사용하는 타입의 메시지를 읽고 쓰는 메소드를 가지고 있다.

- `bool SerializeToString(string* output) const`
  - 메시지를 직렬화하고 전달해준 스트링에 저장한다.
  - 저장된 데이터들은 텍스트가 아닌 바이너리이다.
  - 단지 편리한 컨테이너 사용을 위해서 스트링 클래스를 사용할 뿐이다.
- `bool ParseFromString(const string& data)` : 전달해 준 스트링으로부터 메시지를 파싱한다.
- `bool SerializeToOstream(ostream* output) const` : 전달해 준 C++ ostream 객체로 메시지를 쓴다.
- `bool ParseFromIstream(istream* input)` : 전달해 준 C++ istream에서 메시지를 파싱한다.

> 모든 메소드 리스트 : [Message API reference](https://developers.google.com/protocol-buffers/docs/reference/cpp/google.protobuf.message#Message)

### 주의

> `Protocol Buffers and O-O Design`
>
> 프로토콜 버퍼 클래스들은 기본적으로 dumb 데이터 저장소이다(C++에서 struct 처럼).
>
> 프로토콜 버퍼 클래스들은 오브젝트 모델에서 좋은 일급 객체(first citizen class)들은 아니다.
>
> 생성된 클래스에 좀더 좋은 behaviour를 추가하려면, 가장 좋은 방법은 생성된 클래스를 어플리케이션 클래스로 래핑하는 것이다.
>
> 그리고 .proto 파일의 디자인 전체를 컨트롤하는 것이 없다면, 프로토콜 버퍼를 래핑하는 것은 좋은 생각이다.
>
> 이 경우, 자신의 어플리케이션의 유니크한 환경에 적절한 인터페이스를 구현한 래퍼 클래스를 사용할 수 있다
>
> (몇몇 데이터들과 매소드들은 숨기고 편리한 함수들을 노출시키는 등).
>
> 하지만 절대로 프로토콜 버퍼 클래스들을 상속해서 매소드들을 추가해서는 안 된다.
>
> 이는 내부 메카니즘을 깨는 것이고 좋은 방법이 아니다.

***

## References

- <https://developers.google.com/protocol-buffers/docs/overview>
