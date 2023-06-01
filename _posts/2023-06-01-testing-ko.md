---
layout: post
title: Testing 국문
categories: [testing, SUB]
tag: [testing-ko]
published: false
---

## 개요

쓸데없어 보이지만 나중에 참고할 Markdown 문법들. 한글 테스트 겸용!

# 헤더 1

본문1

## 헤더 2

본문2

### 헤더 3

본문3

#### 헤더 4

본문4

##### 헤더 5

본문5

###### 헤더 6

본문6

### 본문

본문. 이 글씨를 **강조** 해주세요.

![avatar](/assets/img/avatar.jpg)

{: .image-right}

이 글씨 또한 _강조_ 해주세요.

### 인용구

> Hello, world! (반갑구나, 세상아!) - 테카르타 젠야타

## 리스트

### 순서 리스트

1.  아이템 1
    1.  하위 아이템 1
    2.  하위 아이템 2
    3.  하위 아이템 3
2.  아이템 2

### 비 순서 리스트

-   아이템 1
-   아이템 2
-   아이템 3

## 표

| 헤더 1 | 헤더 2 | 헤더 3 |
| :-- | :-: | --: |
| 셀 1 | 셀 2 | 셀 3 |
| 셀 4 | 셀 5 | 셀 6 |
| \---- |   |   |
| 셀 1 | 셀 2 | 셀 3 |
| 셀 4 | 셀 5 | 셀 6 |
| \===== |   |   |
| 풋 1 | 풋 2 | 풋 3 |
| {: rules="groups"} |   |   |

## 코드 스니펫

마크다운 문법

```css
#container {  
float: left;
margin: 0 -240px 0 0;
width: 100%;
}
```

하이라이트 적용

{% highlight css %}  
#container {  
float: left;  
margin: 0 -240px 0 0;  
width: 100%;  
}  
{% endhighlight %}

## 버튼

`.btn` class를 이용해 버튼을 만들자.

{% highlight html %}  
[Success Button](#)  
{% endhighlight %}

[중요한 버튼](#)

[매우 중요한 버튼](#)

[오렌지 버튼](#)

[크고 붉은 버튼](#)

[시시한 버튼](#)

## 키보드

`<kbd>` tag를 사용하여 키보드 버튼을 표시하자.

{% highlight html %}

WASD

  
{% endhighlight %}

Alt

+

F4

를 눌러 댓글을 다십시오. **펑!!**

## 공지

**중요 공지!** `{: .notice}`를 사용하여 해당 단락을 공지로 만들자.  
{: .notice}