---
title: C++20 기능 정리 - Range
date: 2023-11-28 10:47:00 +0900
categories: [Stack, C++]
tag: [C++, C++20, Range]
---

## Range

영문 그대로 번역하면 `범위` 입니다.

어떤 리스트, 배열, 벡터 등 컨테이너나 범위를 지정할 수 있는 무언가가 존재할 때 내가 필요한 만큼만 `보겠다` 는 개념으로 접근하면 이해가 쉬울 듯 합니다.

### Range 기초 코드

[가이드 문서](https://en.cppreference.com/w/cpp/ranges/range)에 따르면 `range` 키워드를 통해 내가 적용할 인스턴스에 범위를 지정할 수 있는지 알 수 있습니다.

```cpp
int              elem1 = 1;
std::vector<int> elem2 = { 1, 2, 3, 4, 5 };

std::cout << std::ranges::range<decltype(elem1)> << std::endl;  // false
std::cout << std::ranges::range<decltype(elem2)> << std::endl;  // true
```

## View

C++20에서 ranges 적용을 위하여 view 개념이 추가되었습니다. 이것을 어떻게 이해하는게 좋을까 고민해봤는데, 어릴 적 한번 쯤 가지고 놀았던 셀로판지와 유사했습니다.

예를 들어 내가 1 부터 5까지 가지는 범위에서 홀수 값만 취하고 싶을 때, 이것만 볼 수 있도록 범위를 지정하면, 이 것이 view로서 저장됩니다.(`output_view`)

```cpp
auto view = elem2 | std::views::filter([](int i) { return i % 2 != 0; });
for (auto i : view)
  std::cout << i << std::endl;
```

조금 더 자세히 알아보면, std::views를 통해 반환 값을 설정하면 `std::views::__adaptor::_RangeAdaptor`의 반환 값이 설정됩니다.
즉 범위 어뎁터는 `view`를 생성하며 이러한 view를 생성하기 위해서 우리는 3가지 방법을 사용할 수 있습니다.

```cpp
std::ranges::filter_view view1(elem2, [](int i) { return i % 2 != 0; });
auto view2 = std::ranges::filter_view(elem2, [](int i) { return i % 2 != 0; });
auto view3 = elem2 | std::views::filter([](int i) { return i % 2 != 0; });
```

사용자마다 다르겠으나 저는 gstreamer, 리눅스에서 사용하는 파이프라인 방식에 익숙하기 때문에 마지막 방법이 가장 사용하기 편해 보였습니다.

> 런타임 속도 관련 벤치마킹 테스트 시 각 방법에 유의미한 차이는 없었습니다.

## 함수의 특이점

가이드 문서를 보면 아래와 같이 `ranges::foo_view` 방식과 `views::foo` 방식이 존재합니다.

![대체 함수](/assets/img/post/2023-11-28-stdcpp-20-range/foos.png)

얼핏 보면 다양한 방식을 지원하기 때문에 편해보일 수 있으나 두 방법의 차이점이 궁금해졌습니다.

구글링을 하다보면 아래 인용을 볼 수 있었습니다.

> The use of `std::views::transform` here is valid, you do not have to write out `std::ranges::views::transform`. The standard library provides `std::views` as a namespace alias for `std::ranges::views` - as you can see in the [synopsis](https://eel.is/c++draft/ranges.syn) for `<ranges>` (towards the end).

인용에 따르면 `views::foo` 방법은 사용자 지향 알고리즘이며, `ranges::foo_views`보다 선호되어야 한다고 주장합니다. 예를들어 views 객체를 const로 생성했을 때, int& 형식의 view의 경우, const inst& 객체의 뷰를 할당합니다.

그러나 이미 const int&를 전달하면 이 뷰는 초기 뷰를 반환합니다. 따라서 일반적으로 전자의 방법이 후자의 방법보다 더 많은 선택을 사용자에게 제공합니다.

> Always prefer views::meow over ranges::meow_view, unless you have a very explicit reason that you specifically need to use the latter - which almost certainly means that you’re in the context of implementing a view, rather than using one.

결론적으로 View를 생성하는 3가지 방법 중, view3을 생성하는 방법을 사용하거나, 또는 아래와 같이 뷰 객체를 직접 할당하는 방법을 사용하는 것이 선호됩니다.

```cpp
auto view4 = std::views::filter(elem2, [](int i) { return i % 2 != 0; });
```

### 예제

마지막으로 유용하게 사용할 수 있는 대-소문자 변환 코드를 ranges를 통해 구현해 보겠습니다.

```cpp
std::string str = "Hello World !\n", str2;

// case1. 대문자로 변환하여 str에 바로 저장하고 싶은 경우
auto up_view1 = str | std::views::transform([](char c) { return std::toupper(c); });
std::ranges::copy(up_view1, str.begin());

// case2. 대문자로 변환하여 str2에 저장하고 싶은 경우
std::ranges::copy(up_view1, std::back_inserter(str2));

// (option) 기존 C++ 방식
std::transform(str.begin(), str.end(), str.begin(), [](char c) { return std::tolower(c); });
std::transform(str.begin(), str.end(), std::back_inserter(str2), [](char c) { return std::tolower(c); });
```

## 저장소

[https://github.com/lasiyan/code-partition/tree/master/stdcpp-20-range](https://github.com/lasiyan/code-partition/tree/master/stdcpp-20-range)
