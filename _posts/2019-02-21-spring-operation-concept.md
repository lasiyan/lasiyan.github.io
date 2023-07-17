---
title: 스프링 개념과 동작 원리 - Spring
date: 2019-02-21 00:44:00 +0900
categories: [Stack, Web]
tag: [웹개발, Web, Spring]
---


## 스프링 개념

프레임워크는 개발자에게 구조화된 기본 뼈대를 제공함으로서 기본 구조를 정하고 이 구조 위에서 코드를 작성하게 함으로서 기본적인 코드를 작성하는 시간 낭비를 줄여줍니다.

스프링 프레임워크 역시 `DI`를 사용하여 디자인 패턴에 대한 자세한 이해가 없어도 객체를 싱글턴 패턴으로 사용할 수 있도록 만들어 줍니다.

> 싱글턴 : 인스턴트가 하나 뿐인 특별한 객체. 어디서든 접근 가능한 객체

스프링 프레임워크는 스프링 컨테이너라는 런타임 엔진을 제공하며 이 컨테이너는 생성에서 소멸까지 객체의 라이프 사이클을 관리하며 이 과정에서 `IoC`와 `DI`를 사용합니다.

## 스프링 용어

`IoC`는 Inversion of Control의 약자로 말 그대로 프로그램의 제어 흐름 구조가 바뀌는 것을 의미합니다. IoC를 통하여 객체의 제어 권한이 스프링 컨테이너에 존재하기 때문에 스프링에서는 일반적인 흐름과 다르게 객체가 언제 생성되고 사용되는지 알 수 없습니다. 따라서 IoC를 사용하면 설계가 깔끔해지고 유연성이 증가하며 확장성이 좋아집니다.

`DI`는 Dependency Injection의 줄임말로 의존성 주입을 의미하고 IoC의 대표적 동작 원리 중 하나입니다. 컨테이너에 의해 객체를 사용할 수 있도록 생성 후 생성자나 setter 함수와 같은 메서드를 통해 사용하는 것을 의미합니다.

스프링 컨테이너에 의해 생성되고 제어되는 객체를 `Bean` 이라 하며 이것은 스프링 컨테이너 내부의 빈(Bean) 팩토리 객체를 통해 싱글턴 패턴으로 생성되고 제어됩니다. 그러나 일반적으로 빈(Bean)팩토리보다 이것의 기능을 모두 포함하고 있는 애플리케이션 컨텍스트를 사용합니다. 애플리케이션 컨텍스트는 빈(Bean)팩토리를 확장한 것으로 빈의 생성과 제어 뿐만 아니라 스프링이 제공하는 어플리케이션 지원 기능을 모두 관리하는 것을 의미합니다.

서버 환경에서 수많은 요청을 받아 처리해야 하는데 그 때마다 객체를 생성하면 비효율적이므로 싱글턴 패턴으로 생성된 빈(Bean) 객체를 통해 이러한 요청을 처리합니다.

`AOP`는 Aspect Oriented Programming의 약어로 기존의 비즈니스 로직 외 작성해야 하는 코드를 별도로 분리함으로써 개발자가 좀 더 비즈니스 로직에만 집중하여 처리할 수 있는 방법을 제공합니다. 예를들어 인터넷 쇼핑몰을 구현할 때 핵심적인 비즈니스 로직은 상품 구매와 장바구니, 구매 목록 조회 같은 기능이지만 이 모든 것은 로그인 기능이 전제되어야 합니다. 따라서 로그인 기능과 핵심 기능(상품 구매, 구매 목록 조회, 장바구니) 기능을 분리하여 핵심 기능에 몰두할 수 있게 하며 그 외 부가적인 개발이나 영향을 미치는 객체로 부터 결합도를 최소화합니다.

`MVC` 패턴이란 모델, 뷰, 컨트롤러의 약자로 이 세가지를 각각 분리하여 웹 개발을 하는 방식입니다. 모델은 데이터를 의미하고 뷰는 사용자에게 보여지는 화면(HTML, JSP 등), 컨트롤러는 비즈니스 로직과 모델, 뷰의 중재자 역할을 합니다.

스프링 프레임워크는 MVC패턴에서 약간의 변형을 주어 `Front Controller`란 것을 만들고 모든 흐름을 이것을 통하여 각각의 Controller로 위임하여 처리합니다. 예를 들어 물건을 A/S하기 위해 대표 전화로 상담전화를 걸 때 전화를 받은 상담원이 우선 고객이 필요한 것이 무엇인지 파악 후 관련 부서로 전화를 돌려줍니다. 바로 이 전화를 가장 먼저 받는 상담원을 Front Controller라 할 수 있습니다.

Front Controller 패턴은 공동 개발에서 규격화된 코드를 작성할 수 있고, 공통된 코드를 각각의 컨트롤러가 아닌 front Controller에서 한 번만 작업하면 되므로 개발 시간도 크게 단축할 수 있습니다.

`Dispatcher Servlet`은 Front Controller 디자인 패턴을 표현한 것입니다.

`Handler Mapping`은 사용자의 각각의 요청에 대한 URL을 비교해 등록된 URL과 일치하는 컨트롤러를 탐색 후 전달하며 컨트롤러는 이 요청을 처리한 후 데이터를 사용자에게 뷰를 통하여 제공합니다.

`Controller`는 전달된 요청을 처리하고 처리 후 결과 데이터를 요청에 맞는 뷰로 전달하는 부분입니다. 하지만 일반 MVC 패턴의 컨트롤러와 다르게 스프링의 컨트롤러는 사용자의 요청에서 필요한 데이터를 보다 쉽게 추출할 수 있고 추출하는 처리를 자동으로 해주기 때문에 개발 시간을 크게 단축시킬 수 있습니다. 또한 기존의 XML 파일을 통해 설정하는 것과 다르게 스프링에서는 애노테이션을 통해 간편하게 제어하므로 컨트롤러의 설정이 간편합니다.

`View Resolver`는 컨트롤러가 처리한 데이터를 각각의 해당하는 뷰에 매핑시켜주는 것을 담당합니다. 스프링 컨트롤러는 View에 의존적이지 않기 때문에 컨트롤러는 뷰의 이름만 지정해주면 뷰 리솔버에서 해당하는 뷰 객체를 생성합니다. 그리고 이렇게 생성된 뷰를 통해 사용자는 원하는 데이터를 제공받을 수 있게 됩니다.

## 동작 과정

![spring_image](https://github-production-user-asset-6210df.s3.amazonaws.com/135001826/253458180-6f75a6ad-8ae5-42d4-ab6f-104534544046.png)

위 그림에서는 최소 사용자의 request 흐름이 Front-Controller인 Dispatcher-Servlet에 전달되면 Dispatcher-servlet은 Handler-Mapping에 request를 보내 해당하는 요청의 URL과 일치한 컨트롤러 URL을 탐색하고 그 정보를 통해 컨트롤러와 연결시킵니다. 요청에 맞는 해당 컨트롤러는 흐름에서 필요한 데이터를 추출하여 요청을 처리하고 다시 Dispatcher-Servlet에 처리된 데이터와 뷰에 관한 정보를 전달합니다. 이러한 정보는 View-Resolver에게 전달되어 View-Reslover는 처리된 데이터에 해당하는 뷰 객체를 생성하여 데이터를 출력하고 이것은 최종적으로 사용자에게 response 됩니다.

## 요약

1. 사용자의 요청이 입력된다.
2. `DispatcherServlet`을 통해 처음으로 전달 받고 `HandlerMapping`을 통해 해당하는 `Controller`와 연결한다.
3. `Controller`는 요청에서 필요한 데이터를 추출 후 해당 데이터에 대한 `View` 이름 지정한다.
4. `ViewResolver`는 데이터에 대한 `View` 객체 생성 후 사용자에게 요청에 대한 데이틀 제공한다.