---
title: DLL에서 데이터를 전달하는 방법 (Notify)
date: 2020-08-06 13:26:55 +0900
categories: [Stack, C++]
tag: [MFC, OnNotify]
---

윈도우 환경에서 프로그램을 개발하다 보면 자연스럽게 DLL을 사용하게 되고, 종종 DLL에서 특정 데이터를 전달하여 해당 값을 사용하는 경우가 생깁니다.

아래 예제는 명시적 방법으로 DLL을 로드하고, DLL에서 Notify를 통해 데이터를 전달하여 메인 프로그램에서 해당 값을 출력하는 예제입니다.

### 개요
프로그램 구조와 전반적인 동작 루틴은 아래와 같습니다.

```
- Common
  - NotifyTestCommon : EXE와 DLL에서 공통으로 사용되는 구조체 또는 전처리문 정의 (.h)
- Execute
  - NotifyTestExeDlg : EXE 메인 UI (.h, .cpp)
- DLL
```

![dll-notify-test](/assets/img/post/2020-08-06-mfc-notify-example/dll-notify.png)


### 명시적 링킹을 위한 선언
```c++
// NotifyTestCommon.h
#ifndef __cplusplus
extern "C" {
#endif

__declspec(dllexport) void DLL_Load(PFXNOTIFYLISTENER pfxNotifyListener);
__declspec(dllexport) void DLL_Test(int type);

#ifdef __cplusplus
}
#endif
```

```c++
// NotifyTestExeDlg.h
typedef	void(*pfxDllFunc_Load)  (PFXNOTIFYLISTENER);
typedef	void(*pfxDllFunc_Test)  (int);

HINSTANCE   m_hDLL;
```

### Notify 구현
```c++
// NotifyTestCommon.h
// ... 이전 구현 내용

typedef	LRESULT		(*PFXNOTIFYLISTENER)	(UINT, WPARAM, LPARAM);

#define ID_NOTIFY_TEST_INTEGER    (111)
#define ID_NOTIFY_TEST_CSTRING    (222)
#define ID_NOTIFY_TEST_CLASS      (333)

enum { eTest_Integer, eTest_CString, eTest_Class };

// 테스트 클래스
class TestClass
{
public:
    int	a;
    BYTE	b;
    char	c;
};
```

```c++
// NotifyTestExeDlg.h
static LRESULT  OnNotify(UINT nID, WPARAM wParam, LPARAM lParam);


// NotifyTestExeDlg.cpp
LRESULT CNotifyTestExeDlg::OnNotify(UINT nID, WPARAM wParam, LPARAM lParam)
{
    CString strMsg;

    switch (nID)
    {
    case ID_NOTIFY_TEST_INTEGER:
        strMsg.Format(_T("%d"), wParam);
        AfxMessageBox(strMsg);
        break;

    case ID_NOTIFY_TEST_CSTRING:
        strMsg.Format(_T("%s"), *((CString*)lParam));
        AfxMessageBox(strMsg);
        break;

    case ID_NOTIFY_TEST_CLASS:
        strMsg.Format(_T("wParam : %d\r\n"),		wParam);
        strMsg.AppendFormat(_T("a : %d\r\n"),		((TestClass*)lParam)->a);
        strMsg.AppendFormat(_T("b : 0x%02X\r\n"),	((TestClass*)lParam)->b);
        strMsg.AppendFormat(_T("c : %c"),		((TestClass*)lParam)->c);

        AfxMessageBox(strMsg);
        break;

    default:
        break;
    }

    return 0;
}
```

```c++
// NotifyTestDLL.h
PFXNOTIFYLISTENER   m_pfxNotifyListener;


// NotifyTestDLL.cpp
void CNotifyTestDLLApp::Notify_Initialize(PFXNOTIFYLISTENER pfxArgument)
{
    m_pfxNotifyListener = pfxArgument;
}

void CNotifyTestDLLApp::NotifyTest_Integer()
{
    int nResult = 12345;

    m_pfxNotifyListener(ID_NOTIFY_TEST_INTEGER, (WPARAM)nResult, NULL);
}

void CNotifyTestDLLApp::NotifyTest_CString()
{
    CString strResult;
    strResult.Format(_T("Notify_Test"));

    m_pfxNotifyListener(ID_NOTIFY_TEST_CSTRING, NULL, (LPARAM)&strResult);
}

void CNotifyTestDLLApp::NotifyTest_Class()
{
    int nResult = 123;

    TestClass testClass;

    testClass.a = 456;
    testClass.b = 0xFF;
    testClass.c = 'A';

    m_pfxNotifyListener(ID_NOTIFY_TEST_CLASS, nResult, (LPARAM)&testClass);
}
```
Notify Messasge는 기본적으로 특정 컨트롤 윈도우에서 발생시킨 각종 이벤트를 부모 윈도우에서 처리하는 목적으로 사용합니다.

본 예제에서는 Notify Message를 구분하기 위한 ID를 정의하고 Exe 소스코드에서 해당 ID를 통해 DLL로부터 전달 받은 데이터를 처리합니다.

처리부는 OnNotify에서 진행되며, 해당 함수의 포인터를 DLL 생성(로드) 시 전달합니다.


### DLL 로드 및 테스트
```c++
// NotifyTestDLL.cpp
void DLL_Load(PFXNOTIFYLISTENER pfxNotifyListener)
{
    AFX_MANAGE_STATE(AfxGetStaticModuleState());
    theApp.Notify_Initialize(pfxNotifyListener);
}

void DLL_Test(int type)
{
    switch (type)
    {
    case eTest_Integer:
        theApp.NotifyTest_Integer();
        break;

    case eTest_CString:
        theApp.NotifyTest_CString();
        break;

    case eTest_Class:
        theApp.NotifyTest_Class();
        break;

    default:
        break;		
    }
}
```
DLL은 메인 프로그램(EXE)에서 전달 받은 함수 포인터를 저장하고(`m_pfxNotifyListener`) 이를 통해 메인 프로그램에서 DLL_Test 함수를 호출 시 값을 전달합니다.

```c++
// NotifyTestExeDlg.cpp
BOOL CNotifyTestExeDlg::OnInitDialog()
{
    // ...

    m_hDLL = ::LoadLibrary("MyDLL.dll");

    if (m_hDLL != NULL)
    {
        pfxDllFunc_Load	DllFunc_Load = (pfxDllFunc_Load) ::GetProcAddress(m_hDLL, _T("DLL_Load"));

		    if (DllFunc_Load)
			      DllFunc_Load(OnNotify);
    }
    else
    {
      AfxMessageBox(_T("Failed to Load DLL"));
    }

    // ...
}

// 예시
void CNotifyTestExeDlg::OnBnClickedBtnNotifyTestCString()
{
    pfxDllFunc_Test	DllFunc_Test = (pfxDllFunc_Test) ::GetProcAddress(m_hDLL, _T("DLL_Test"));

    if (DllFunc_Test)
        DllFunc_Test(eTest_CString);
}
```
