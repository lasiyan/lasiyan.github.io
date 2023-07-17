---
title: 윈도우 폴더 강제 삭제
date: 2022-02-21 13:26:55 +0900
categories: [Stack, C++]
tag: [MFC]
---

MFC 기본 함수 중 RemoveDirectory() 함수는 삭제하려는 폴더 안에 파일 또는 다른 폴더가 존재할 경우 FALSE를 반환합니다.

> GetLastError: 145 (The directory is not empty.)

이를 해결하기 위해 MSDN에서는 SHFileOperation 함수 사용을 제안하고 있습니다.

> To recursively delete the files in a directory, use the [SHFileOperation](https://learn.microsoft.com/en-us/windows/win32/api/shellapi/nf-shellapi-shfileoperationa){:target="_blank"} function.

아래 코드는 위 방법 대신 더 간결한 코드를 활용해서 폴더를 재귀적으로 지우는 방법입니다.

```cpp
void CClientControl::DeleteDirectory_Shell()
{
	CString strCmd;
    
	CString strTarget;
	// strTarget.Format(삭제할 폴더 경로 + 이름);
	// strTarget.Format(_T("E:\\Working\\test_folder")); 예시
	
	strCmd.Format(_T("/c echo Y | cmd /c rd /s %s"), strTarget);
	::ShellExecute(NULL, "open", "cmd", (LPCSTR)strCmd, NULL, SW_HIDE);
}
```

단, rd 명령어는 실행할 때 확인을 위해 응답(Y)을 받는데, `echo Y`를 통해 이를 강제로 동작 시킵니다.

> [위험한 커맨드 TOP 10](https://www.tecmint.com/dangerous-linux-commands/){:target="_blank"}

*따라서 반드시 동작 전 strTarget에 정상적인 경로가 들어가는지 확인하고, 절대 경로를 사용합니다.*
