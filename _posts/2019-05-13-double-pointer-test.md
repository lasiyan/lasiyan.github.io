---
title: C언어 더블 포인터 정리
date: 2019-05-13 12:54:13 +0900
categories: [Stack, C++]
tag: [C언어, Pointer]
---

## 포인터와 주소

### 포인터
- 메모리의 주소 값을 담고 있는 변수 또는 상수
- 데이터 위치를 가리키는 변수

### 주소
- 메모리 저장된 위치(번지)
- `&` 연산자를 사용하면 특정 변수의 주소를 반환

## 단항 간접 참조 연산자
단일 포인터

```c
#include <stdio.h>

void main()
{
    int a;
    int *pa;
     
    a=10;
    pa=&a;
     
    printf("a: %d\n", a);  // 10
    *a=100;
    printf("a: %d\n", a);  // 100
}
```

## 더블 포인터
```c
#include <stdio.h>

void main()
{
    int A = 50, B = 100;
    int* pA = &A;
    int** dpA = &pA;
    
    /* Test 1 */
    printf("pA : %d\n", *pA);  // 50
    printf("dpA : %d\n\n", **dpA);  // 50

    /* Test 2 */
    *dpA = &B;
    printf("pA : %d\n", *pA);  // 100
    printf("dpA : %d\n\n", **dpA);  // 100

    /* Test 3 */
    **dpA = 999;
    printf("B : %d\n", B);  // 999
    printf("pA : %d\n", *pA);  // 999
    printf("dpA : %d\n\n", **dpA);  // 999
    
    /* Test 4 (Maybe compile error) */
    **dpA = &A;  // address of A (ex. 11223344)
    printf("pA : %d\n", *pA);  // 11223344
    printf("dpA : %d\n\n", **dpA);  // 11223344
}
```

위 결과를 더블 포인터를 초점으로 정리해보면,

1. 더블 포인터 형으로 선언한 변수 dpA는 pA가 가리키는 변수의 주소를 가리킨다.
2. 1에 따라 *dpA는 dpA가 가리키는 포인터 pA의 값을 의미한다.
3. 따라서 *dpA에 B의 주소를 대입하는 것은 pA에 B의 주소를 대입하는 것과 동일하다.
4. 결과적으로 포인터의 값을 변경하는 것은 포인터가 가리키는 변수의 값을 변경하는 것과 동일하다.


