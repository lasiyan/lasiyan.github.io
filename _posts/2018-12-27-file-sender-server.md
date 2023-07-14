---
title: WinSock File Transfer with TCP (Server)
date: 2018-12-27 13:31:00 +0900
categories: [etc., Utility]
tag: [C++, WinSock, TCP, Network, Server]
---

### TCP 기반의 서버 클라이언트 파일 전송 프로그램 (서버 파트)

서버 부분에 프로젝트 생성 후 아래 코드가 위치한 곳에 전송할 파일을 두고 서버부 실행

사용 포트 : 9090

```cpp
/* Server */ 
#pragma comment (lib,"ws2_32.lib")
#include <stdio.h> 
#include <io.h> 
#include <fcntl.h> 
#include <winsock2.h> 
#include <Windows.h> 
#include <time.h> 
 
#define BUFSIZE 65536 
#define PORT 9090 
 
void errorPrint(char* message) { 
    LPVOID lpMsgBuf; 
    FormatMessage( 
        FORMAT_MESSAGE_ALLOCATE_BUFFER | FORMAT_MESSAGE_FROM_SYSTEM,
        NULL, WSAGetLastError(),
        MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT), 
        (LPTSTR)&lpMsgBuf, 0, NULL); 
    printf("[%s] %s", message, (char *)lpMsgBuf); 
    LocalFree(lpMsgBuf); 
}
 
 
HANDLE hFile; 
HANDLE hFileMapping; 
 
void WINAPI dataSending(LPVOID arg) { 
    SOCKET dataSock = (SOCKET)arg; 
    SOCKADDR_IN dataAddr; 
    int dataAddrSz = sizeof(dataAddr); 
    int retval;
  
    getpeername(dataSock, (SOCKADDR*)&dataAddr, &dataAddrSz); 
 
    DWORD dwFileSizeHigh; 
    __int64 qwFileSize = GetFileSize(hFile, &dwFileSizeHigh); 
    __int64 qwFileOffset = 0; 
 
    while (qwFileSize > 0) { 
        DWORD dwBytesInBlock = BUFSIZE; 
        if (qwFileSize < BUFSIZE) { 
            dwBytesInBlock = qwFileSize; 
        }
  
        char* pbFile = (char*)MapViewOfFile
        (
            hFileMapping,
            FILE_MAP_READ,
            (DWORD)(qwFileOffset >> 32),  // 상위 오프셋
            (DWORD)(qwFileOffset & 0xFFFFFFFF),
            dwBytesInBlock
        );
 
        // 파일전송
        retval = send(dataSock, pbFile, dwBytesInBlock, 0);
        if (retval != dwBytesInBlock) { 
            while (1) { 
                retval = send(dataSock, pbFile, dwBytesInBlock, 0); 
                if (retval != 0) break; 
            } 
        }
 
        // 뷰를 다 썼으므로, 뷰를 해제한다. 
        UnmapViewOfFile(pbFile); 
 
        // 오프셋 및 남은 파일 크기 갱신 
        qwFileOffset += dwBytesInBlock; 
        qwFileSize -= dwBytesInBlock; 
    } 
}
 
int main() { 
    srand((unsigned)time(NULL));  
    char fname[256] = { NULL }; 
    while (1) { 
        printf("Sending File name : "); 
        gets_s(fname);
 
        hFile = CreateFile 
        ( 
            TEXT(fname),   // 파일명 
            GENERIC_READ,   // 읽기 
            0, 
            NULL, 
            OPEN_EXISTING, 
            FILE_FLAG_SEQUENTIAL_SCAN, 
            NULL 
        ); 
        if (hFile == INVALID_HANDLE_VALUE) { 
            puts("There is no file"); 
            continue; 
        } 
        else break; 
    }
 
    // 파일 크기 구하기 
    unsigned int fileSize = GetFileSize(hFile, NULL); 
    printf("Sending file Size : %d bytes\n\n", fileSize); 
 
    // WinSock 초기화 
    WSADATA wsa; 
    if (WSAStartup(MAKEWORD(2, 2), &wsa) != 0) return -1; 
 
    // Socket 생성 
    SOCKET mainSock; 
    if ((mainSock = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP)) == INVALID_SOCKET) { 
        errorPrint("Create Socket Error"); 
        return -1;
    }
 
    // 생성한 소켓에 주소 정보를 설정한다.
    SOCKADDR_IN mainSockAddr;
    ZeroMemory(&mainSockAddr, sizeof(mainSockAddr)); 
    mainSockAddr.sin_family = AF_INET; 
    mainSockAddr.sin_addr.s_addr = htonl(INADDR_ANY); 
    mainSockAddr.sin_port = htons(PORT); 
 
    // Bind - 소켓에 주소를 할당한다. 
    int retval; 
    if ((retval = bind(mainSock, (SOCKADDR*)&mainSockAddr, sizeof(mainSockAddr))) == SOCKET_ERROR) 
        errorPrint("Socket Bind Error"); 
 
    // 파일 전송 소켓 옵션 설정 
    int sendBuf, sendBufSize = sizeof(sendBuf); 
    if ((retval = getsockopt(mainSock, SOL_SOCKET, SO_SNDBUF, (char*)&sendBuf, &sendBufSize)) == SOCKET_ERROR) 
        errorPrint("getSockOpt Error"); 
    printf("Before send buffer size : %d bytes\n", sendBuf); 
 
    sendBuf = BUFSIZE; 
    if ((retval = setsockopt(mainSock, SOL_SOCKET, SO_SNDBUF, (char*)&sendBuf, sendBufSize)) == SOCKET_ERROR) 
        errorPrint("getSockOpt Error"); 
 
    if ((retval = getsockopt(mainSock, SOL_SOCKET, SO_SNDBUF, (char*)&sendBuf, &sendBufSize)) == SOCKET_ERROR) 
        errorPrint("getSockOpt Error"); 
    printf("After send buffer size : %d bytes\n", sendBuf);
  
    // 소켓을 대기상태로 설정한다. 
    retval = listen(mainSock, 3); 
    if (retval == SOCKET_ERROR) { 
        errorPrint("Socket Listen Error"); 
    } 
 
    // 클라이언트의 접속을 기다림 
    puts("\nWaiting for Client...\n"); 
 
    // Accept - 클라이언트 연결을 받아들임 = 클라이언트와 연결 될 소켓 생성 
    SOCKET sizeSock, dataSock; 
    SOCKADDR_IN sizeAddr, dataAddr; 
    int sizeAddrSz = sizeof(sizeAddr); 
    int dataAddrSz = sizeof(dataAddr); 
    while (1) { 
        if ((sizeSock = accept(mainSock, (SOCKADDR*)&sizeAddr, &sizeAddrSz)) == INVALID_SOCKET) 
            errorPrint("Accept Error"); 
        else break; 
    } 
    while (1) { 
        if ((dataSock = accept(mainSock, (SOCKADDR*)&dataAddr, &dataAddrSz)) == INVALID_SOCKET) 
            errorPrint("Accept Error"); 
        else break; 
    }
 
    printf("Client Connect from IP : %s, PORT : %d-%d\n\n", 
        inet_ntoa(dataAddr.sin_addr), ntohs(sizeAddr.sin_port), ntohs(dataAddr.sin_port)); 
 
    // 파일 크기 전송 
    retval = send(sizeSock, (char*)&fileSize, sizeof(fileSize), 0); 
 
    // send - 파일 데이터 전송 시작 
    puts("File Data Sending Start"); 
 
    // 파일 매핑 
    hFileMapping = CreateFileMapping 
    ( 
        hFile,   // 맵 시킬 파일의 핸들 
        NULL,   // 보안 설정 
        PAGE_READONLY,   // 파일속성과 맞춤 
        0,      // dwMaximumSizeHigh 
        0,    // dwMaximumSizeLow 
        NULL 
    ); 
 
    // 수신 스레드 생성 
    DWORD hDataThreadID; 
    HANDLE hDataThread = CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)dataSending, (SOCKET*)dataSock, 0, &hDataThreadID); 
    WaitForSingleObject(hDataThread, INFINITE);   
    puts("File Sengind Finish"); 
    shutdown(dataSock, SD_SEND);
  
    // 클라이언트에서 받은 파일 사이즈 
    unsigned int sendSize; 
    if ((retval = recv(dataSock, (char*)&sendSize, sizeof(sendSize), 0)) == SOCKET_ERROR) 
        errorPrint("Recv Respone Message Error"); 
 
    // 클라이언트 사이즈와 원본 사이즈 비교 - 같으면 성공, 다르면 에러 
    if (sendSize != fileSize) puts("File Data Sending Error"); 
    else puts("File Data Sending Complete\n"); 
 
    CloseHandle(hFileMapping); 
    CloseHandle(hFile); 
    closesocket(dataSock); 
    closesocket(mainSock); 
    WSACleanup();
    return 0; 
}
```


