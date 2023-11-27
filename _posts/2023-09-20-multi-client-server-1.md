---
title: 다중 클라이언트 서버 - 1
date: 2023-09-20 14:22:12 +0900
categories: [Stack, Network]
tag: [C++, Server, "서버"]
---

다중 클라이언트가 접속하는 서버를 구현하는 베이스 코드입니다.

기존 1:1 방식이 아닌, N 명의 클라이언트에 대하여 select를 활용하여 각 클라이언트의 요청을 처리합니다.

> 현재 진행 중인 웹 브라우저와 리눅스 기반 응용 프로그램 사이, 중계 서버를 만들기 위하여 테스트 한 코드입니다.

---

### 개요

먼저, 일반적으로 알고 있는 `socket`의 개념과 별개로 select에 대하여 간략히 설명하자면, `select`는 네트워크 프로그래밍에서 사용되는 함수 중 하나로,
여러 개의 소켓을 모니터링하고 입출력 가능한 상태를 감지하는 기능을 제공합니다. 이를 통해 동시에 여러 소켓을 관리하고 비동기 네트워크 통신을 구현할 수 있습니다.

일반적으로 `select`는 유닉스 기반의 운영체제(Linux, mac, FreeBSD 등)에서 제공되는 함수입니다. 하지만 Windows 역시 select 함수 대신 WSASelectEvent 함수 등을 사용하여 이와 비슷한 기능을 구현할 수 있습니다.

확장성 및 여러 포트에서 각각 다른 메세지 처리를 위해 베이스 서버 클래스를 파생하여 여러개의 서버를 생성하도록 설계하였습니다.

> 하기 모든 코드는 포스팅를 위해 간소화한 코드입니다

### 코드 설명

1. Server

   먼저 베이스로, 모든 서버에서 공통으로 사용되는 변수 및 함수를 정의합니다.

   대표적으로 소켓 정보, 소켓 생성, 소켓 종료와 서버 시작 및 종료가 있습니다.

   ```cpp
   class Server {
     virtual void running() = 0;  // 서버 루프 메서드

    public:
     Server() = default;
     virtual ~Server();

    public:
     virtual bool start();
     virtual void stop();

    protected:
     // 소켓 제어
     bool openSocket();
     void closeSocket();
  
    protected:
     int socket_   = MY_INVALID_SOCKET;
   };
   ```

2. MultiClientServer

   Server 클래스를 상속 받아 생성되는 상위 클래스입니다. 이 클래스는 클라이언트의 접속을 수락하여 내부 리스트에 저장하고, 클라이언트 요청을 처리합니다.

   ```cpp
   class MultiClientServer : public Server {
     void running() override;
   
    public:
     virtual ~MultiClientServer() { stop(); }
   
     const std::vector<ConnectedClient*>& clients() const { return client_list_; }
     const ConnectedClient*               client(int socket);

    public:
     void stop() override;

    private:
     virtual int onReceived(const ConnectedClient* client_info) = 0;
   
    protected:
     std::vector<ConnectedClient*> client_list_;
   };
   ```

3. TestServer

   마지막으로 실제 서버 구현을 위한 메인 클래스로, 이번 포스트는 설명을 위해 간단하게 에코 서버를 생성합니다.

   실제 프로젝트에서는 OPCode 값에 따라 `onReceived()`에서 메세지를 처리하는 방식으로 진행되는데, 이는 다음 포스트에서 진행합니다.

   본 서버는 싱글턴 클래스로 프로그램 종료 시 서버 객체도 같이 삭제됩니다. 지연 생성이 적용되어 있으므로, 생성 시점은 `isntance()`가 호출되는 시점입니다.


   ```cpp
   class TestServer : public server::MultiClientServer {
    public:
     static TestServer& instance() {
       static TestServer instance_;
       return instance_;
     }
   
    private:
     TestServer() {
       port_ = 9999;
     }
   
     int onReceived(const ConnectedClient* sClient) override;
   };
   ```

---
```cpp
  // initialize socket
  fd_set fd_reads;
  int    fd_max, fd_num;

  while (thread_stop_.load() == false) {
    FD_ZERO(&fd_reads);

    FD_SET(socket_, &fd_reads);
    fd_max = socket_;
    
    for (unsigned int i = 0; i < client_list_.size(); i++) {
      fd_num = client_list_[i]->client_sock;
      FD_SET(fd_num, &fd_reads);
      if (fd_num > fd_max)
        fd_max = fd_num;
    }

    timeval select_timeout = { 1, 0 };

    int activity = select(fd_max + 1, &fd_reads, NULL, NULL, &select_timeout);
    if (activity < 0) {
      printf("select() : error : %d : %s\n", errno, strerror(errno));
      break;
    }

    if (activity == 0) {
      usleep(10);
      continue;
    }
    ...
```
먼저 `fd_set`는 일종의 소켓을 담는 컨테이너라고 볼 수 있습니다. 따라서 우리는 소켓을 `fd_reads`에 담아서 변화를 감지하고 처리하는데, 이 과정을 `select` 함수가 담당합니다.

서버 메인 스레드가 시작되면 먼저 서버 소켓을 fd_reads에 추가하고, `fd_max`에 할당합니다. 참고로 `fd_max`는 현재 관리 중인 소켓들 중 가장 큰 `File Descriptor`를 의미합니다.
> File Descriptor: OS에서 열린 파일, 소켓 또는 파이프와 같은 I/O 리소스를 식별하는 값

`select` 함수는 입력 대기 중인 소켓을 확인하고 결과 값을 반환하는데, 값이 음수인 경우, 네트워크 통신 또는 소켓 세트 이상으로, 스레드를 종료합니다.

또한 블로킹 방지를 위하여 timeout 값을 설정하는데, timeout 시 0이 반환되므로, 필요한 다른 작업을 처리하고(ex. 서버 종료 확인) 다시 대기 확인 상태로 진입합니다.

---
```cpp
    for (int i = 0; i < client_list_.size(); i++) {
      int client_socket = client_list_[i]->client_sock;

      if (FD_ISSET(client_socket, &fd_reads)) {
        auto sClient = client(client_socket);

        if (sClient) {
          try {
            int ret = onReceived(sClient);
            ...
```
`FD_ISSET`은 파일 디스크립터 집합(`fd_set`)에서 특정 파일 디스크립터가 활성화되어 있는지를 확인하는 함수로, `select` 함수와 함께 사용하여 어떤 소켓이 활성화되었는지 합니다. `select` 함수는 활성화된 소켓들의 목록을 `fd_set`에 설정하고, 이후 `FD_ISSET`을 사용하여 개별 소켓의 활성화 여부를 확인합니다.
> 다중 클라이언트 서버와 같은 환경에서 여러 소켓을 관리하면서, 어떤 소켓에서 데이터가 도착했는지 또는 연결 요청이 도착했는지를 확인할 때 사용됩니다.

메인 소켓, 다시 말해 `socket_`의 변화를 FD_ISSET으로 감지하며 클라이언트의 접속 여부를 판단하며, 각 클라이언트 소켓을 FD_ISSET으로 감지하며 메세지 수신 여부를 확인합니다.

만약 클라이언트로부터 0을 수신 받는 경우는 클라이언트와 연결이 종료된 것을 의미하므로 클라이언트 리스트에서 해당 소켓을 삭제합니다.
```c
/* Read NBYTES into BUF from FD.  Return the
   number read, -1 for errors or 0 for EOF. */
extern ssize_t read (int __fd, void *__buf, size_t __nbytes) __wur;
```

---

```cpp
int EchoServer::onReceived(const ConnectedClient* client)
{
  std::unique_lock<std::mutex> ulock(message_sync_mutex_);

  uint8_t buffer[PACKET_BUFFER_SIZE];
  memset(buffer, '\0', sizeof(buffer));

  int read_size = read(client->client_sock, buffer, sizeof(buffer));

  if (read_size <= 0) {
    return read_size;
  }

  // 전송 받은 메세지를 다시 클라이언트로 전송합니다
  if (sendMessage(client->client_sock, buffer, read_size) == read_size) {
    printf("echo : %s\n", buffer);
  }

  return read_size;
}
```

마지막으로 각 서버는 pure virtual method인 `onReceived` 함수를 통해 수신된 메세지를 처리합니다.

본문에서는 수신된 메세지를 클라이언트로 재전송 하는 기능을 구현합니다.

다음 포스트에서는 실무에서 사용하는 프로젝트들과 유사하게 간단한 Packet을 구성해서 OP-Code에 따른 메세지 처리를 구현할 예정입니다.

### 참고

원본 코드: [https://github.com/lasiyan/code-partition/tree/master/multi-client-server-1](https://github.com/lasiyan/code-partition/tree/master/multi-client-server-1)