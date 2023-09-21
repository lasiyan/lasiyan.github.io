---
title: 다중 클라이언트 서버 - 2
date: 2023-09-21 13:25:00 +0900
categories: [Stack, C++]
tag: [C++, Server, "서버"]
---

> 지난 글: [다중 클라이언트 서버 - 1](https://lasiyan.github.io/posts/multi-client-server/)

이번 포스트는 지난번 다중 클라이언트 처리를 위해 `select`를 활용하여 만든 서버에 `Packet` 개념을 추가한다.

### 패킷 구조

아래 패킷은 일반적으로 통신을 함에 있어 가장 간단한 패킷 형식 중 하나이다.

![image](https://github.com/lasiyan/lasiyan.github.io/assets/135001826/c6c7ae60-5af8-434f-8766-86e46a6cefda)

패킷은 크게 Header와 Payload(Body)로 나뉜다. 물론 이 프로젝트에 한에서이다.

`Header`는 이번에 클라이언트가 보낸 메세지가 어떤 목적인지 나타내는 Command와 뒤따라 오는 Payload의 크기를 나타내는 DataSize가 필요하다.

이를 선언하면 다음과 같다

```c
struct PacketHeader {
  int command;  // Protocol OP-Code (4 byte)
  int datasize; // Packet Data Payload size (4 byte)
}
```

다음으로 `Payload`는 실제 데이터가 담긴 위치이다.

본문에서는 일반 unsigned char 크기의 1바이트 데이터 형식을 사용하지만, 실제로 JSON 또는 YAML 등 다양한 형식이 사용될 수 있다.

### 커맨드 선언

먼저 아래와 같이 protocol.h 파일이 추가되었다. 여기서 패킷과 사용할 OP-Code 목록을 정의한다.

```cpp
enum OP_CODE {
  CMD_ECHO = 1,

  CMD_SET_MESSAGE,
  CMD_GET_MESSAGE,
};
```

기존에 구현했던 에코 기능과 더불어, 이번에는 클라이언트가 특정 메세지를 전송하면 이것을 저장했다가, 다시 요청하면 반환하는 코드를 작성한다.

### 메세지 처리

```cpp
int MyServer::onReceived(ConnectedClient* client)
{
  std::unique_lock<std::mutex> ulock(message_sync_mutex_);

  // variables
  unsigned char* body   = nullptr;
  PacketHeader*  header = nullptr;

  int read_size = recvMessage(client->client_sock, header, body);

  ...

  // 데이터 처리
  switch (header->command) {
    case CMD_ECHO: {
      auto res = sendMessage(client->client_sock, body, header->datasize);
      if (res == header->datasize) {
        printf("echo : %s\n", body);
      }
      break;
    }
    case CMD_SET_MESSAGE:
      snprintf(client->client_message, sizeof(client->client_message), "%s", body);
      break;
    case CMD_GET_MESSAGE: {
      auto res = sendMessage(client->client_sock,
                             reinterpret_cast<uint8_t*>(client->client_message),
                             strlen(client->client_message));
      if (res == strlen(client->client_message)) {
        printf("response : %s\n", body);
      }
      break;
    }
    default:
      break;
  }
  ...
}
```

먼저 기존과 눈에 띄게 달라진 점은 siwtch-case문이 추가되었다.

이것은 헤더 패킷의 커맨드에 따라 서버가 어떤 작업을 실행할지 결정한다. 여기에 각 커맨드 별로 처리하고 싶은 코드를 추가할 수 있다.

이전 기술한 바와 같이 메세지를 수신 받으면 각 클라이언트 내부에 하나의 변수(client_message)를 생성하고 여기에 값을 저장 후, GET 요청에서 이를 반환한다.
> 물론 이를 위해 ConnectedClient* 의 const를 제거했습니다.

### 메세지 수신

또한 이전 코드와 다르게 `recvMessage` 함수가 추가되었다.

이것은 수신 받은 데이터에서 Header와 Payload를 분리(파싱)하는 역할을 담당한다. 여기서 주의 깊게 처리되어야 하는 것은 아래와 같다.

- 데이터에서 PacketHeader가 존재하는가? (데이터의 크기가 8바이트 이상인가?)
  - command가 유효한 범위인가?
  - datasize가 정상적인 길이인가?

> 아래 코드는 파싱 부분을 처리한 예시입니다.

```cpp
int MyServer::recvMessage(int socket, PacketHeader*& header, uint8_t*& buffer)
{
  uint8_t read_buffer[Server::PACKET_BUFFER_SIZE];
  bzero(read_buffer, sizeof(read_buffer));

  auto ret = read(socket, read_buffer, sizeof(read_buffer));

  // wrong message, so drop
  if (ret < sizeof(PacketHeader))
    return ret;

  // Parse Header & Data
  header = new PacketHeader;
  memcpy(header, read_buffer, sizeof(PacketHeader));

  buffer = new uint8_t[header->datasize + 1];  // 1 : eos
  memset(buffer, '\0', header->datasize + 1);  // add null
  memcpy(buffer, read_buffer + sizeof(PacketHeader), header->datasize);
  ...
}
```

따라서 정상적인 데이터를 수신할 경우 그 Payload의 데이터가 client_message에 저장되고, 이를 반환한다.

### 맹점

코드를 테스트 하다 보면 한 가지 치명적인 문제가 발생한다.

기존 코드에서 나는 `PACKET_BUFFER_SIZE`를 8192 바이트로 설정하였다.

그런데 만약.. 수신 받은 크기가 이 것보다 클 경우 데이터 손실이 발생한다

먼저, 테스트를 위해 `PACKET_BUFFER_SIZE`를 12 바이트로 변경하였다. 그럼 헤더 패킷의 크기가 8 바이트 고정이므로, 나는 총 4자리의 문자열을 받을 수 있다.
만약 여기서 5자리 이상의 문자열을 입력할 경우, 데이터 손실이 발생하는 것이다.

```
// 실행 결과 (에코, 수신: "12345")
echo : 1234
192.168.0.241 : invalid header or size (read: 1)  // 마지막 1바이트가 누락됨
```

따라서 아래 코드와 같이 수정하여 Header의 datasize에 입력된 값 만큼 데이터가 수신될 때까지 대기하고, 이를 처리하는 코드로 변경한다.

```cpp
int MyServer::recvMessage(ConnectedClient* sClient, PacketHeader*& header, uint8_t*& buffer)
{
  constexpr int MAX_RETRY = 3;

  bool recv_buffer_over = false;  // PACKET_BUFFER_SIZE보다 큰 값을 수신한 경우

  int recv_error_count = 0;  // 수신 에러 카운트
  int total_recv_size  = 0;  // 수신된 총 데이터 크기
  int total_data_size  = 0;  // 수신된 총 데이터 크기 (헤더 포함)
  int remain_data_size = 0;  // 수신 받아야 할 잔여 데이터 크기

  uint8_t read_buffer[Server::PACKET_BUFFER_SIZE];
  memset(read_buffer, '\0', sizeof(read_buffer));

  int ret = 0;

  while ((ret = read(sClient->client_sock, read_buffer, sizeof(read_buffer))) != 0) {
    try {
      // ret가 0 이하인 경우( 에러가 발생한 경우 )
      if (ret <= 0) {
        recv_error_count++;
        if (ret == 0)  // 연결 종료 (연결 종료는 에러가 아니다)
          throw false;
        else if (recv_error_count >= MAX_RETRY)
          throw true;
        else
          continue;
      }

      // 에러 카운트 초기화
      recv_error_count = 0;

      // 총 수신 크기 저장
      total_recv_size += ret;

      // 추가 수신(패킷 버퍼 오버플로) 상태가 아닌데, 헤더보다 작은 경우 예외처리
      if (recv_buffer_over == false && ret < sizeof(PacketHeader))
        break;

      // 추가 수신(패킷 버퍼 오버플로) 상태가 아닌 경우
      if (recv_buffer_over == false) {
        header = new PacketHeader;
        memcpy(header, read_buffer, sizeof(PacketHeader));
        int curr_data_size = ret - sizeof(PacketHeader);
        total_data_size += curr_data_size;
        remain_data_size = header->datasize - curr_data_size;

        // 커맨드가 유효한지 확인
        if (header->command < CMD_ECHO || header->command > CMD_GET_MESSAGE) {
          printf("invalid command code: %d\n", header->command);
          break;
        }

        buffer = new unsigned char[header->datasize + 1];  // 1 : eos
        memset(buffer, '\0', header->datasize + 1);
        memcpy(buffer, read_buffer + sizeof(PacketHeader), curr_data_size);

        // 수신된 데이터가 헤더의 데이터 크기보다 작은 경우 ( 추가 수신 필요 )
        if (header->datasize > curr_data_size) {
          recv_buffer_over = true;
          continue;
        }
        else {
          break;
        }
      }
      else {
        // 추가 수신(패킷 버퍼 오버플로) 상태인 경우
        if (remain_data_size < ret)  // buffer의 overflow 방지
          ret = remain_data_size;

        remain_data_size -= ret;

        memcpy(buffer + total_data_size, read_buffer, ret);
        total_data_size += ret;

        // 총 수신된 데이터가 헤더의 데이터 크기보다 큰 경우
        if (total_data_size > header->datasize) {
          printf("buffer overflow.. some of received data is missing\n");
          break;
        }
        // 총 수신된 데이터가 헤더의 데이터 크기와 같은 경우
        else if (total_data_size == header->datasize) {
          break;
        }
        // 총 수신된 데이터가 헤더의 데이터 크기보다 작은 경우
        else {
          continue;
        }
      }
    }
    // .. catch 에러 처리
  }
  return total_recv_size;
}
```

Header의 datasize를 기준으로 수신 받아야 하는 데이터 크기를 가변적으로 조절한다.

여기서 데이터 자체가 최대 수신할 수 있는 버퍼의 크기(`PACKET_BUFFER_SIZE`)를 초과하는 데이터가 수신될 경우
데이터 누락 방지를 위해 `recv_buffer_over` 플래그를 `true`로 변경하고 잔여 데이터를 수신 받는다.


### 저장소

원본 코드: [https://github.com/lasiyan/code-partition/tree/master/multi-client-server-2](https://github.com/lasiyan/code-partition/tree/master/multi-client-server-2)
