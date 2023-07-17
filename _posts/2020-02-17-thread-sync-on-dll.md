---
title: 쓰레드를 사용한 동기화 경험 (with. DLL)
date: 2020-02-17 13:26:55 +0900
categories: [Stack, C++]
tag: [MFC, Thread, Synchronize, 동기화]
---

일반적으로 두 프로그램 간 동기화 중 가장 간단한 방법은 while문을 통해 A 프로그램에서 전달한 신호가 B 프로그램에서 인식될 때 까지 기다린 후, 원하는 커맨드가 전달되면 프로그램을 실행하는 방법이 있습니다.

그러나 위 방법으로 개발을 진행할 때, A 프로그램에서 커맨드를 전달하는 부분이 많은 경우, 또는 B 프로그램에서 하나의 신호가 아닌, 다중 신호에 대한 처리를 진행할 때, while문에 막혀, 다른 프로세스가 처리되지 않을 수 있습니다.

아래 코드는 B 프로그램에서 while문을 사용했을 때, while문 루틴에 걸려 다른 코드가 실행되지 않음을 경험하고, 별도 Thread와 WaitForSingleObject를 통해 동기화를 처리했던 코드입니다.

```c++
// header
static bool    g_Sync;  // 해제 신호
static int     g_CurrentSuspend;
static HANDLE  g_ThreadSync;

DWORD WINAPI ThreadSyncronize(LPVOID param);

void WaitFor(int eCmd);	// eCmd를 기다린다.
void Resume(int eCmd);	// WaitFor 해제
```
```c++
// source
#define DEF_SUSPEND_TIMEOUT (20000)
DWORD WINAPI ThreadSyncronize(LPVOID param)
{
  // 1. 대기를 시작한 시간을 저장
  DWORD dwStart = ::GetTickCount();
  while(TRUE)
  {
    // 20초 이상 기다리는 경우 : Timeout
    if(::GetTickCount() - dwStart > DEF_SUSPEND_TIMEOUT)
      break;

    // 해제 신호가 수신된 경우
    if(g_Sync)
      break;
  }

  return (DWORD)TRUE;
}

CCriticalSection g_cs;

void WaitFor(int eCmd)
{
  // eCmd는 일종의 Keyword로,
  // Resume시 해당 Key가 아닐 경우 동기화가 해제되지 않도록
  g_cs.Lock();	// WaitFor을 동시에 호출하는 부분이 많을 경우 필요.
  g_CurrentSuspend = eCmd;
  g_Sync = false;
 
  if(g_ThreadWait == NULL)
  {
    DWORD dwResult;
    g_ThreadWait = CreateThread(NULL, 0, ThreadSyncronize, NULL, 0, &dwResult);
    g_cs.Unlock();

    // while문과 같이 해당 부분에서 기다린다.
    WaitForSingleObject(g_ThreadWait, INFINITE);
  }
  else
  {
    // 1개 이상의 WaitFor 함수가 호출될 경우,
    // 이미 CS로 g_ThreadWait은 생성되어 있기 때문에, WaitForSingleObject만 진행
    g_cs.Unlock();
    WaitForSingleObject(g_ThreadWait, INFINITE);
  }

  g_cs.Lock();

  // Thread 종료.
  if(g_ThreadWait != NULL)
  {
    ::TerminateThread(g_ThreadWait, NULL);
    CloseHandle(g_ThreadWait);
    g_ThreadWait = NULL;
  }

  g_cs.Unlock();
}

void Resume(int eCmd)
{
  // WaitFor에서 지정된 Keyword(eCmd)가 아닐 경우 무시
  if(g_CurrentSuspend != eCmd)
    return;
  g_Sync = true;
}
```