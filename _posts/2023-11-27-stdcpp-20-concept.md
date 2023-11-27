---
title: C++20 기능 정리 - Conecpt
date: 2023-11-27 22:27:00 +0900
categories: [Stack, C++]
tag: [C++, C++20, Conecpt]
---

## 제약 조건

[cplusplus.com](https://en.cppreference.com/w/cpp/language/constraints)(이하. 가이드 문서)에 따르면 Concept을 다음과 같이 설명하고 있다.

> Named sets of such requirements are called concepts. Each concept is a predicate, evaluated at compile time, and becomes a part of the interface of a template where it is used as a constraint:

쉽게 말하면 템플릿에 포함되는 인자(인수)들에 대한 요구 사항의 집합을 `concept` 이라 명칭하고, 각 개념은 컴파일 시점에 측정되어 템플릿 인터페이스의 일부로 동작된다는 의미이다.

더욱 쉽게 말하면 내가 원하는 인자만 template 인자로 쓸 수 있도록 제약을 걸겠다는 의미이다.

### 예시 1

가이드 문서는 Hash를 예시로 들고 있다.

```cpp
#include <string>
#include <cstddef>
#include <concepts>

template<typename T>
concept Hashable = requires(T a)
{
    { std::hash<T>{}(a) } -> std::convertible_to<std::size_t>;
};
 
struct meow {};
 
// Constrained C++20 function template:
template<Hashable T>
void f(T) {}

int main()
{
    using std::operator""s;
 
    f("abc"s);    // OK, std::string satisfies Hashable
    // f(meow{}); // Error: meow does not satisfy Hashable
}
```

위 코드에서 abc는 hash 생성이 가능하지만, meow 구조체 인스턴스는 hash 변환이 불가능하기 때문에 제약 사항을 걸어준 것이다.

실제로 meow 인스턴스를 인자로 컴파일 시 아래와 같은 로그가 출력된다.

```
[build] /home/user/code-partition/example-of-cpp-20/main.cpp: In function ‘int main()’:
[build] /home/user/code-partition/example-of-cpp-20/main.cpp:38:4: error: no matching function for call to ‘f(meow)’
[build]    38 |   f(meow{});  // Error: meow does not satisfy Hashable
[build]       |   ~^~~~~~~~
[build] /home/user/code-partition/example-of-cpp-20/main.cpp:19:6: note: candidate: ‘template<class T>  requires  Hashable<T> void f(T)’
[build]    19 | void f(T)
```

기존 에러들과 다르게, 매우 친절하게 에러 내용을 보여준다.

### 예시 2

아래 코드는 기존 template 인자로 float 또는 double 가능하도록 제약 조건을 설정한 코드이다.

```cpp
template <typename T>
typename std::enable_if<std::is_floating_point<T>::value>::type
foo(T t) {}

int main()
{
  foo(1.0f);
  // foo(1);
}
```

이 코드에 concepts을 적용하면 아래와 같이 더욱 간단한 코드가 가능하다.

```cpp
#include <concepts>

template <std::floating_point T>
void foo(T t) {}

// Is also, same above
// template <typename T> requires std::floating_point<T>
// void foo(T t) {}
```

## 더 알아보기

물론 `concept`을 단순히 타입에만 접목하면 코드를 보다 간결하게 만든다는 것에 지나지 않는다.

하지만 이전 포스트와 같이 concept이 주요 변경점 top4에 포함된 이유는 C++ 20의 concept은 조건을 지정하고 그 조건에 해당하지 않은 경우 컴파일 시점에 오류를 발생시킨다는 것이다.

더 쉽게 알아보자.

### 예시

예를들어 다음과 같은 구조체가 있다. 이것은 a라는 변수를 담고 있는 구조체로, 이것을 상속 받아 STRUCT_X 와 STRUCT_Y 자식 구조체를 선언한다.

```cpp
struct STRUCT { int a = 1; };
struct STRUCT_X : STRUCT { int x = 2, z = 3; };
struct STRUCT_Y : STRUCT { int y = 2, z = 3; };
```

그리고 다음과 같이 부모 구조체를 통해 함수를 호출하는데, 만약 템플릿 인자에 x라는 변수가 존재하지 않을 경우 컴파일 시점에 에러를 발생시키고 싶다.

```cpp
template <typename T>
void foo(T t) {
  t.x;
}

int main()
{
  STRUCT_X sx;
  STRUCT_Y sy;
  foo(&sx);
  foo(&sy);
}
```

가장 간단한 방법은 T의 인스턴스 t.x를 호출하면 되지만 이것은 컴파일 시점에 다음과 같이 에러의 위치가 한 눈에 파악하기 힘들다. (예시는 코드가 짧기 때문에 파악이 가능하지만 실제로 상속된 클래스가 수십개라면.. 끔찍하다)

물론 `declval` 와 `if constexpr` 를 조합하면 사전에 체크가 가능하고, 다른 방법 역시 불가능한 것은 아니지만, C++ 20에서 아래와 같이 제약 조건을 설정하고,

이 조건을 논리 연산하여 더욱 쉽게 인자에 대한 제한이 가능해진다.

```cpp
template <typename T>
concept HasX = requires(T t) { t.x; };

template <typename T>
concept HasZ = requires(T t) { t.z; };

template <typename T>
concept HasVar = HasX<T> && HasZ<T>;
void foo(HasVar auto t) {}

int main()
{
  STRUCT_X sx;
  STRUCT_Y sy;
  foo(sx);
  foo(sy);
}
```
```
main.cpp: Candidate template ignored: constraints not satisfied [with T = STRUCT_Y]
main.cpp: Because 'STRUCT_Y' does not satisfy 'HasX'
```

친절하게 HasX 조건이 만족되지 않았기 때문에, foo(sy) 라인에서 에러가 발생한다는 것을 보여준다.

### 종합 코드

아래 코드는 concept에 대하여 다양한 예시를 직관적으로 알 수 있게 해주는 예시 코드이다.

```cpp
template <typename T>
concept Decrementable = requires(T t) { --t; };
template <typename T>
concept RevIterator = Decrementable<T> && requires(T t) { *t; };

template <Decrementable T>
void f(T t) { std::cout << "#1" << " " << t << "\n"; }  // #1

template <RevIterator T>
void f(T t) { std::cout << "#2" << " " << t << "\n"; }  // #2

template <class T>
void g(T t) { std::cout << "#3" << " " << t << "\n"; }  // #3

template <Decrementable T>
void g(T t) { std::cout << "#4" << " " << t << "\n"; }  // #4

template <typename T>
concept RevIterator2 = requires(T t) { --t; *t; };

template <Decrementable T>
void h(T t) { std::cout << "#5" << " " << t << "\n"; }  // #5

template <RevIterator2 T>
void h(T t) { std::cout << "#6" << " " << t << "\n"; }  // #6

int main()
{
  f(0);        // #1 실행
  f((int*)0);  // #2 실행

  g(true);  // 
  g(0);     // 

  h((int*)0);  // ambiguous

  return 0;
}
```

한 줄씩 살펴보자. 먼저 `f(0)`는 `f` 함수 중 `Decrementable` 조건과 `RevIterator` 중 RevIterator 조건을 만족하지 않는다. RevIterator 조건은 역참조 연산을 만족해야 하는데, 0에 대한 역참조는 불가하기 때문이다. 다시말해 f 함수의 인자 x가 있을 때, x가 0인 경우 `--` 또는 `++` 연산은 가능하지만, `*0` 연산은 불가하다는 것이다.

반대로 f 함수의 인자 x를 `int*` 형으로 캐스팅한 값 `x2`는 증감연산 역시 가능하며 포인터에 대한 역참조(= 0) 역시 가능하기 때문에 `#2`가 호출된다.

g 함수를 보면, #3 함수의 경우 일반적인 클래스로 모든 인자를 전부 수용할 수 있는 템플릿 함수이다. 반면 #4 함수의 경우 #1, #2 와 마찬가지로 증감연산이 가능한 값만 인자로 취급되는데, C++에서 bool 타입은 선행 증감 연산자가 불가능하고, 또한 역참조 역시 불가능하기 때문에 `g(true)` 는 #3 함수가 호출된다. 

마지막으로 h 함수를 보면, #5 함수는 선행 증감 연산자 조건이 만족하는 경우이며, #6 함수는 선행 증감 연산자가 만족하면서 역참조 역시 만족하는 경우를 의미한다.

그런데 `h((int*))` 함수는 main 코드의 #2 라인과 동일하게 증감 연산 및 역참조 연산이 모두 가능한데, 컴파일러 입장에서 #5 와 #6 중 어떤 함수를 호출할지 결정할 수 없기 때문에 모호하다(ambiguous)는 에러를 발생시킨다.

## 결론

결과적으로 우리는 C++ 20에서 새롭게 추가된 `requires` 구문을 통해 템플릿 함수(클래스)의 인자에 조건을 걸고, 일치하는 조건에 대한 템플릿 함수를 호출할 수 있는 방법이 가능하다는 것을 알 수 있었다.

템플릿을 `타입` 관점에서 주로 접근하면서 개발을 해왔기 때문에 반드시 써야 하는 유의미한 상황을 생각하긴 어렵지만 `조건`을 제외하고라도 보다 간결한 템플릿 구현이 가능해진 다는 점이 코드의 가독성 향상 측면에서 도움을 줄 수 있을 것 같다.
