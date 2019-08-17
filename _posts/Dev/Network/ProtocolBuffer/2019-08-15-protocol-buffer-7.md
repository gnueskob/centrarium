---
layout: post
title:  "Google protocol buffer (7) with C++"
date:   2019-08-17 22:03:00 +0900
author: Gnues
categories: network
tags:	network protocol-buffer google cpp
---

이번 포스팅에서는 protobuf를 사용한 예제를 살펴본다.

***

## Network Packet buffer

이번에 소개할 예제에서는 프로토콜 버퍼를 네트워크 라이브러리에서 사용하는 예제이다.

네트워크상 주고받는 패킷들이 완전한 상태로 오지 않는 경우가 있기 때문에 이를 저장해두는 공간이 필요하다.

때문에 이를 저장해두고 완전한 패킷 사이즈만큼 데이터를 받으면 처리하게 된다.

예제의 네트워크 라이브러리에서는 byte 배열의 패킷 버퍼에 프로토콜 버퍼를 이용해서 직렬화(Serialize)하여 쓰고 읽는다.

***

## 예제 .proto message

간단히 로그인, 로그아웃이 가능한 서버가 있다고 가정하자.

아래 proto message는 login, logout 기능별 요청, 응답 패킷에 대응되는 message이다.

echo는 그대로 돌려주기 때문에 하나의 message로만 사용할 것이다.

```proto
syntax = "proto3";
package lsbProto;

// Echo packet
// Request and Response format are same
message Echo {
  string msg = 1;
}

/**********************************/

// Login Packet
message LoginReq {
  string id = 1;
  string pw = 2;
}

message LoginRes {
  int32 res = 1;
}

/**********************************/

// Logout Packet
message LogoutReq {
}

message LogoutRes {
  int32 res = 1;
}

/**********************************/
```

이 **.proto** 파일을 protoc로 컴파일하여 생성된 클래스를 프로젝트에 포함시켜 사용한다.

아래 설명의 proto 클래스란 .proto 파일의 메시지를 컴파일하여 생성된 클래스를 의미한다.

***

## Message class

예제 코드를 소개하기에 앞서 프로토콜 버퍼에 존재하는 **Message** 클래스를 소개한다.

**.proto**파일을 통해 생성된 클래스는 모두 **::PROTOBUF_NAMESPACE_ID::Message** 클래스를 상속한다.

```cpp
// .proto 파일로 생성한 'message Echo'의 클래스 예시
class Echo :
    public ::PROTOBUF_NAMESPACE_ID::Message /* @@protoc_insertion_point(class_definition:lsbProto.Echo) */ {
 public:
  Echo();
  virtual ~Echo();

  ...
```

**.proto** 파일에서 여러가지 메시지를 정의하고 각 메시지마다 클래스를 생성시키지만, 모두 **Message**로 캐스팅하여 처리할 수 있다.

***

## 예제 서버의 패킷 처리

클라이언트로부터 패킷이 들어왔다고할 때 서버에서는 다음과 같은 순서를 거쳐 패킷을 처리한다.

1. 서버로 패킷이 전달되고 패킷버퍼에 패킷이 저장된다.
2. 패킷 헤더에 정의된 패킷 body크기 만큼 충분한 양의 패킷이 들어왔다면 한 단위의 요청이 들어왔다고 판단한다.
3. 요청 패킷에 대한 처리를 하기위해 패킷 버퍼(byte배열)에서 패킷 body크기만큼 데이터를 가져온다.
4. 패킷을 처리한다.
5. 처리 완료후 결과를 다시 반대의 순서로 응답 패킷을 만들어 클라이언트에게 보낸다.

흔히 처리되는 클라-서버 간의 네트워크 프로세스이다.

우리가 지금 주목해야 할 부분은 **(3)**번째 단계의 패킷버퍼의 데이터 역직렬화 부분과 **(5)**번째의 직렬화 부분이다.

***

## Proto class

**C++**로 만들어진 네트워크 라이브러리 코드는 다음과 같다.

```cpp
using namespace google::protobuf;

...

ERROR_CODE PacketProcess::Login(PacketInfo packet)
{
   lsbProto::LoginReq reqPkt;
   auto pReqProto = dynamic_cast<Message*>(&reqPkt);
   ParseDataToProto(pReqProto, packet.pData, packet.PacketBodySize);

   auto err = m_pUserMngr->AddUser(packet.SessionId, reqPkt.id().c_str());

   lsbProto::LoginRes resPkt;
   resPkt.set_res(static_cast<google::protobuf::int32>(err));

   auto packetId = static_cast<short>(PACKET_ID::LOGIN_RES);
   auto pResProto = dynamic_cast<Message*>(&resPkt);

   if (err != ERROR_CODE::NONE)
   {
      m_pLogicMain->SendProto(packet.SessionId, packetId, pResProto);
      return err;
   }

   m_pConnectedUserManager->SetLogin(packet.SessionId);

   m_Log->Write(LV::DEBUG, "%s | Login. id : %s, session Id : %d", __FUNCTION__, reqPkt.id().c_str(), packet.SessionId);
   m_pLogicMain->SendProto(packet.SessionId, packetId, pResProto);

   return ERROR_CODE::NONE;
}
```

위 코드는 로그인 과정의 예제 코드이다.

.proto 파일의 `package lsbProto;` 문구로 namespace가 `lsbProto`임을 확인할 수 있다.

`PacketProcess::Login` 함수에서 벌어지는 일은 다음과 같다.

1. 로그인 요청 패킷 정보를 담을 `lsbProto::LoginReq` reqPkt 인스턴스를 선언

2. `ParseDataToProto` 함수에서는 `Message`와 패킷 포인터, 데이터 크기를 받아 파싱

3. proto(`LoginReq`) 클래스를 통해 서버 로직을 처리
   - `LoginReq`의 ID string 사용
   - `reqPkt.id().c_str()`

4. 응답 패킷을 보내기 위한 `lsbProto::LoginRes` resPkt 인스턴스 선언
   - `LoginRes`의 응답 코드를 세팅
   - `LoginRes.set_res(...)`

5. 로그인 응답 코드 셋팅 `resPkt.set_res(...)`

6. 요청 데이터와 마찬가지로 응답 패킷 클래스(`LoginRes`)를 `Message`로 캐스팅하여 메시지 전송(`SendProto`)

사전에 정의한 Proto 클래스의 멤버 변수를 사용하기 위해서는 (3), (4) 항목처럼 사용하면 된다.

.proto파일에서 정의한 `LoginReq`의 message에서 정의한 `id`, `pw`는 같은 이름의 함수로 얻을 수 있으며 해당 값을 바꾸기 위해선 `set_` prefix를 붙인 함수를 이용한다.

***

## Read from char array to proto class

예제의 네트워크 라이브러리에서는 byte배열(`char` array)로 패킷 버퍼를 구현하였다.

`google::protobuf::io::ArrayInputStream`를 통해 `char`배열로부터 자신이 정의한 Proto 클래스로 역직렬화(Unserialize) 할 수 있다.

위의 예제에서 사용한 `PacketProcess::ParseDataToProto(..)`함수를 자세히 들여다보자.

```cpp
bool PacketProcess::ParseDataToProto(Message* pProto, char* pData, short size)
{
   io::ArrayInputStream is(pData, size);
   return pProto->ParseFromZeroCopyStream(&is);
}
```

- 함수는 3가지를 parameter로 전달받는다.
  - 정의한 proto 클래스(`LoginReq`)의 주소를 `Message*`로 캐스팅한 포인터
  - 읽어들일 데이터가 있는 패킷 버퍼의 위치(포인터)
  - 해당 패킷 버퍼의 위치로부터 읽어들일 데이터의 크기 (bytes 수)

- `google::protobuf::io::ArrayInputStream`를 이용해서 byte 배열로부터 `Message`로 데이터 파싱이 가능하다.

- `ArrayInputStream` 인스턴스에 데이터 위치(`pointer`)와 읽어올 데이터의 크기를 알려준다.

- `Message`의 `ParseFromZeroCopyStream(..)` 멤버함수를 통해 패킷 버퍼데이터를 proto(`LoginReq`) 클래스로 파싱한다.

`Message` 클래스로 업캐스팅하여 사용하지만 패킷 버퍼에 존재하던 데이터를 정의한 클래스의 형태로 파싱할 수 있다.

***

## Write from char array to proto class

응답을 위한 Proto 클래스에 원하는 값을 셋팅했다면 데이터를 직렬화하여 패킷 버퍼에 저장해야한다.

이때 사용하는 `SendProto(..)` 함수를 자세히 들여다보자.

```cpp
void LogicMain::SendProto(const int sessionId, const short packetId, Message* pMsg)
{
  auto bodyLength = static_cast<short>(pMsg->ByteSize());
  auto totalSize = static_cast<short>(PACKET_HEADER_SIZE + bodyLength);
  PacketHeader header{ totalSize, packetId, static_cast<unsigned char>(0) };

  auto err = m_pNetwork->SendPacket(
    sessionId
    , bodyLength
    , nullptr
    , PACKET_HEADER_SIZE
    , reinterpret_cast<char*>(&header)
    , [&pMsg](char* writePos, int bodyLength) -> bool {
      io::ArrayOutputStream os(writePos, bodyLength);
      return pMsg->SerializeToZeroCopyStream(&os);
    });

  if (err != NET_ERROR_CODE::NONE)
  {
    ...
  }
}
```

- 세션 ID, 패킷 ID, Proto 클래스 포인터를 `Message*`로 업캐스팅한 3가지 인자를 받는다.

- 패킷을 보내기 위해 패킷 헤더를 생성한다.
  - 패킷 body의 크기를 알기 위해 proto 클래스의 크기를 확인한다. `pMsg->ByteSize()`
  - 미리 정의한 패킷 헤더의 크기와 합쳐 총 패킷의 크기를 계산한다.
  - 패킷 ID를 추가하여 패킷 헤더를 만든다.
  - 예제에서 사용된 패킷 헤더 구조는 다음과 같다.

    ```cpp
    struct PacketHeader
    {
      short PacketSize;
      short Id;
      unsigned char Reserved;
    };
    ```

- 예제의 네트워크 라이브러리에서는 `SendPacket(..)`함수를 제공하여 패킷을 보낼 수 있다.
  - 일반적으로 보내려는 데이터의 위치(`char *` 포인터)와 크기를 보내면 라이브러리 상에서 자동으로 패킷 버퍼에 데이터를 복사한다.
  - 하지만 사용자가 직접 패킷 버퍼에 데이터를 복사하는 함수를 넘겨주면 해당 함수를 통해 패킷 버퍼에 데이터를 복사한다.

Protobuf를 사용하기 때문에 패킷버퍼에 데이터를 복사하는 별도의 함수를 넘겨줘야 하고 그 부분이 `SendPacket(..)` 함수에 인자로 추가된 람다함수이다.

```cpp
[&pMsg](char* writePos, int bodyLength) -> bool
{
  io::ArrayOutputStream os(writePos, bodyLength);
  return pMsg->SerializeToZeroCopyStream(&os);
}
```

이 람다함수는 proto 클래스 포인터를 캡쳐하고 두 개의 변수를 인자로 받는다.

`char* writePos`는 패킷 버퍼가 가리키는 쓰기가능한 버퍼의 시작점이고 `int bodyLength`는 보내야할 데이터의 크기이다.

네트워크 라이브러리 제공하는 2개의 변수를 통해 패킷 버퍼에 데이터를 복사하는 함수를 만들 수 있다.

Byte 배열 역직렬화는 `google::protobuf::io::ArrayOutputStream`을 통해 할 수 있고 이 때 데이터를 저장하려는 byte 배열의 포인터와 크기가 필요하다.

- 앞서 네트워크 라이브러리가 제공하는 두 개의 인자를 통해 `ArrayOutputStream`인스턴스를 만든다.

- `Message`의 `SerializeToZeroCopyStream(..)` 멤버함수를 통해 proto 클래스의 데이터를 byte 배열에 직렬화한다.

이로써 응답 패킷 proto 클래스의 데이터를 직렬화하여 패킷 버퍼로 저장하게 된다.
