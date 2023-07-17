---
title: C언어 malloc에 대하여
date: 2019-07-26 20:11:37 +0900
categories: [Stack, C++]
tag: [C언어, malloc]
---

## Memory allocation

`malloc`은 코딩에서 동적 메모리 할당이라는 부분을 배울 때 가장 처음 등장하는 단어입니다.

이는 memory allocation의 준말로 정해진 static 공간이 아닌 dynamic한 공간이 프로그램에서 필요할 때 사용합니다.

```c
#include <stdlib.h>
 
int main()
{
	int length;
	char str1[100];
	char *str2;
 
	printf("Input size: ");
	scanf("%d", &length);
 
	str2 = (char*)malloc(sizeof(char) * length + 1);
  // ...
  // 해제
}
```

사용자가 글자를 입력하는 상황이 발생할 때, 위와 같이 str1처럼 정적으로 받는 방식과 str2처럼 실제 필요한 크기만큼 동적으로 할당해서 사용하는 방식이 있습니다.

예를 들어 *abcde*라는 문자열을 입력했을 때 글자 수는 총 5자리, NULL을 포함하여 6byte 크기를 가집니다.

따라서 malloc을 통해 `5+1` 바이트를 할당해서 str2를 사용하면 온전하게 공간을 효율적으로 사용할 수 있지만,

str1을 사용할 경우 총 100 바이트 메모리에서 6바이트만 사용하고, 94바이트는 낭비되는 공간으로 남습니다.

반대로 101 바이트를 str1에 담으려고 하면 `overflow`가 발생합니다.

> 실제로 메모리를 효율적으로 사용해야 하는 임베디드 환경에서 동적 할당을 자주 사요합니다.

> 그러나 항상 malloc이 정답은 아닙니다. 동적 할당은 정적 할당보다 런타임 시 동작 시간이 오래 걸립니다.

---

소스 코드에서 동적할당을 메모리가 실제로 얼마나 사용되는지, 작업관리자를 통해 확인할 수 있습니다.

아래 코드는 총 1Gb의 메모리를 선언하는 코드입니다.

> 메모리 누수가 발생합니다. 테스트 시 강제 종료가 필요합니다.

```c
#include <stdio.h>
#include <stdlib.h>
 
int main()
{
	malloc(1024 * 1024 * 1024);
	while(1);
}
```

프로그램을 실행하면 아래 사진과 같이 작업 관리자에서 내가 실행한 프로그램의 `메모리` 항목에서 1Gb에 근사한 값을 볼 수 있습니다.

![pannel](/assets/img/post/2019-07-26-about-malloc/pannel.jpeg)


---

그렇다면 malloc이 메모리를 할당한다는 것은 알았고,뒤의 ()안에 할당할 크기를 입력한다는 것을 알았습니다.

그럼 앞의 `char*`는 왜 사용하는 것일까요?

---

이는 malloc 함수의 원형에서 알 수 있습니다.

```c
void *malloc(
  size_t size
);
```

malloc 함수는 `void*` 타입을 반환합니다. 이는 다시 말해 사용자가 어떠한 자료형을 사용할지 모르니, 사용할 타입에 따라 변형해서 사용하라는 의미입니다.

주로 이것은 malloc을 해제할 때 사용됩니다.

결론적으로 개발자는 암묵적 룰에 따라 아래와 같이 내가 의도한 자료형의 크기를 할당했으면, 그것에 해당하는 자료형으로 반환 받아서 해제까지 진행하여야 합니다.

```c
// Bad
char* ptr = (int*)malloc(1024);

// Bad
char* ptr = (char*)malloc(1024 * sizeof(int));

// Good
char* ptr = (char*)malloc(1024);

// Good
int* ptr = (int*)malloc(1024 * sizeof(int));
```
실제로 위 4가지 방법은 어느 하나 문제를 발생하지 않습니다. `char`나 `int` 모두 인간이 알아보기 쉬운 `단어`일 뿐 컴퓨터 입장에서 1바이트, 4바이트 그 이상도 이하도 아니기 때문입니다.

하지만 아래 코드와 같이 기계에게 혼란을 주게 되면 일부 컴파일 환경에서 메모리 누수가 발생할 수 있습니다.

> SMT7 SoC 환경에서 테스트 시 메모리 누수 발생

```c
// Good
char* ptr = (char*)malloc(1024);
free((char*)ptr);

// Memory Leak
int* ptr = (char*)malloc(1024);
free((char*)ptr);
```