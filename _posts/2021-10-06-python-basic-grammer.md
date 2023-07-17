---
title: 파이썬 기본 문법 정리
date: 2021-10-06 00:44:00 +0900
categories: [Stack, Python]
tag: [파이썬, Python, 문법]
---

*코테 전 보면 좋은 기본 문법 정리*

## 자료형
정수형, 실수형, 문자열, 리스트, 튜플, 딕셔너리 등

### 정수

```python
num = 100	# 양의 정수
print(num)	# 100

num = 0		# 제로(0)
print(num)	# 0

num = -100	# 음의 정수
print(num)	# -100
```


### 실수
정수형에 `소수점 아래 자리값`이 포함된 자료형

```python
num = 123.12	# 실수
print(num)	# 123.12

num = -123.	# 소수점 아래 0 생략
print(num)	# -123.0

num = 10e3	# 지수형(양의 정수)
print(num)	# 10000.0	(10 x e(10)^3)

num = 123e-3	# 지수형(음의 정수)
print(num)	# 0.123		(123 x e(10)^(-3) == 123 * 0.001)
```

```python
# 0.1 + 0.2 = 0.3 인가?
num = 0.1 + 0.2	
print(num)		# 0.30000000000000004

# 그럼 0.3을 정확하게 표현하는 법
num = round(0.1 + 0.2, 0)	# 소수점 첫 번째(0+1) 자리 반올림
print(num)			# 0.0

num = round(0.1 + 0.2, 1)	# 소수점 두 번째(1+1) 자리 반올림
print(num)			# 0.3

num = round(0.1 + 0.2, 3)	# 소수점 네 번째(3+1) 자리 반올림
print(num)			# 0.3
```


#### 실수형 연산
실수형의 사친연산, 거듭제곱 등

```python
a = 3
b = 5

print(a + b) 	# 8

print(a * b)	# 15 (곱하기)
print(a ** b)	# 243 (거듭제곱 3^5)

# 주의
print(a / b)	# 0.6 (3 나누기 5의 실수형 반환)
print(a % b) 	# 3 (3 나누기 5의 나머지)
print(a // b) 	# 0 (3 나누기 5의 몫)
```


### 리스트
C, Java의 Array 그리고 C++의 Vector와 유사 (가장 자주 쓰이는 자료형 중 하나)

> 리스트는 매개변수로 전달될 때 `복사본`이 아닌 `참조`이므로 외부에서 변경 시 실제 데이터가 같이 변경된다.

```python
# 선언
lst = [1, 2, 3, 4, 5]
lst1 = []
print(lst)		# [1, 2, 3, 4, 5]
print(lst1)		# []

# 선언2
lst2 = [1] * 5		# 값이 1이고, 크기가 5인 리스트 생성
print(lst2)		# [1, 1, 1, 1, 1]

# 원소 참조
print(lst[2])		# 3 (인덱스 2, 다시 말해 리스트 상 3번 째 값)
```

#### 리스트 연산 
자주 사용하는 연산으로 Indexing, Slicing, Comprehension 등이 있다.

```python
lst = [1, 2, 3, 4, 5, 6, 7]

# 리스트 인덱싱
print(lst[2])		# 3 (앞에서 3번째 원소 출력)
print(lst[-4])		# 4 (뒤에서 4번째 원소 출력)
print(lst[3])		# 4 (앞에서 4번째 원소 출력)

# 리스트 슬라이싱	[a:b] 는 "a+1"번째부터 "b"번째까지
print(lst[2:5])		# [3, 4, 5] (3번째 원소부터 "5"번째 원소까지 출력)

# 리스트 컴프리헨션
# 0 ~ 4까지 원소를 가지는 리스트
lst2 = [i for i in range(5)]	# [0, 1, 2, 3, 4]

# 1 ~ 20까지 수 중 홀수만 포함하는 리스트
lst3 = [i for i in range(1, 21) if i % 2 == 1]
print(lst3)			# [1, 3, ... 19]
```

#### 리스트 주요 함수

```python
arr = [3, 4, 5]

# append() : 리스트에 원소를 삽입
arr.append(2)		# [3, 4, 5, 2]

# sort() : 리스트 원소 정렬
arr.sort()		# [2, 3, 4, 5]
arr.sort(reverse=True)	# [5, 4, 3, 2]

# insert() : 특정 인덱스에 원소 삽입
arr.insert(0, 5)	# 인덱스 0에 5 삽입
print(arr)		# [5, 5, 4, 3, 2]

# count() : 리스트 안에 특정 값의 개수
print(arr.count(1))	# 0
print(arr.count(3))	# 1
print(arr.count(5))	# 2

# remove() : 특정 원소 값을 제거
arr.remove(2)
print(arr)		# [5, 5, 4, 3]
arr.remove(5)		# 중복된 값은 아래처럼 1개만 제거된다
print(arr)		# [5, 4, 3]
arr.remove(5)
print(arr)		# [4, 3]
arr.remove(1)		# 없는 원소를 지우려고 하기 때문에 에러 발생
```

```python
## 심화 ##
# 리스트에서 특정 값을 가지는 모든 원소 제거하기

arr = [1, 2, 3, 4, 5, 5, 6]
remove = {2, 5}		# 리스트에서 2와 5는 모두 제거

arr = [i for i in arr if i not in remove]
print(arr)		# [1, 3, 4, 6]

remove = {3, 4, 6}	# 리스트에서 3, 4, 6 모두 제거
arr = [i for i in arr if i not in remove]
print(arr)		# [1]
```


### 문자열
큰 따옴표(`" "`) 또는 작은 따옴표(`' '`)로 표현된 값

```python
# 큰 따옴표, 작은 따옴표 모두 무관하다
str1 = "Hello World"
print(str1)		# Hello World

str2 = 'Hello World'
print(str2)		# Hello World

if str1 == str2:
	print("Same")	# Same 출력
else:
	print("Diff")
```

```python
# 큰 따옴표, 작은 따옴표 모두 무관하다
# 문자열 연산 (더하기)
s1 = 'Hello'
s2 = "World"

print(s1 + ' ' + s2)	# Hello Wrold

# 문자열 연산 (곱하기)
s = "ABC" * 3
print(s)		# ABCABCABC

# 문자열 연산 (슬라이싱)
s = "ABCDEFG"
print(s[1:4])		# BCD (인덱스 1부터 4-1까지)


##### 불가능 #####
# 문자열 연산 (인덱스 변경) 
s = "ABBDEFG"
s[2] = "C"		# 에러 발생
'''
Traceback (most recent call last):
  File "e:\Projects\test.py", line 19, in <module>
    s[2] = "C"
TypeError: 'str' object does not support item assignment
'''
```

### 튜플
리스트와 유사하다. 그러나 한번 선언된 값을 변경할 수 없다.

`enum` 또는 `const vector`와 비슷한 개념


### 딕셔너리
`Key`와 `Value` 쌍으로 이루어진 자료형으로 `Hash Table`을 이용한다. 

알고리즘 풀이 시 가장 자주 사용하는 자료형 중 하나로, STL의 `unordered_map`과 유사하다.

데이터의 조회 및 수정은 `O(1)`에 처리될 만큼 매우 빠르지만 Hash 충돌을 고려해야하며, Key 값은 `Immutable`, 다시 말해 변경 불가능한 자료형만 사용이 가능하다.

> Im-mutable : 정수, 실수, 문자열, 튜플 등  
> Mutable : 리스트, 딕셔너리, Numpy 배열 등

또한 Json 객체를 표현하기 가장 적합하다.

```python
# 딕셔너리 선언1
dic = dict()
dic['A'] = 80
dic['B'] = 70
dic['C'] = 90
print(dic)		# {'A': 80, 'B': 70, 'C': 90}

# 딕셔너리 선언2
dic2 = {
    'A' : 80,
    'B' : 70,
    'C' : 90,
}
print(dic2)		# {'A': 80, 'B': 70, 'C': 90}

# 조회1
print(dic['A'])		# 80

# 조회2
print(dic['D'])		# 에러 발생
'''
KeyError: 'D'
'''

# 조회3
if 'B' in dic:
    print("존재한다")    # 출력
else:
    print("존재하지 않음")
```
```python
dic = dict()
data["수지"] = 80
data["수현"] = 90
data["길동"] = 75

# 키(Key)만 추출
keys = dic.keys()
print(keys)		# dict_keys(['수지', '수현', '길동'])

# 값(Data)만 추출
datas = dic.values()
print(datas)		# dict_values([80, 90, 75])

# 예시
for key in keys:	# 80
	print(dic[key])	# 90
    			# 75
```


### 집합
순서가 없고, 중복을 허용하지 않는 자료형

```python
# 집합 선언1
s1 = set([1, 1, 2, 3, 4, 5, 5, 5])
print(s1)	# {1, 2, 3, 4, 5}

# 집합 선언2
s2 = {1, 1, 3, 3, 5, 5, 5, 7, 7, 9}
print(s2)	# {1, 3, 5, 7, 9}

# 합집합
print(s1 | s2)	# {1, 2, 3, 4, 5, 7, 9}

# 교집합
print(s1 & s2)	# {1, 3, 5}

# 차집합
print(s1 - s2)	# {2, 4}
print(s2 - s1)	# {9, 7}
```

#### 집합 주요 함수

```python
s = set([4, 2, 3])
print(s)	# {2, 3, 4} <- 자동으로 정렬됨

# add() : 새로운 원소 추가
s.add(5)
print(s)	# {2, 3, 4, 5}
s.add(2)	# 중복된 값은 추가되지 않음
print(s)	# {2, 3, 4, 5}

# update() : 원소 여러 개 추가
s.update([8, 5, 7])
print(s)	# {2, 3, 4, 5, 7, 8}

# remove() : 원소 '값' 삭제(인덱스 아님)
s.remove(3)
print(s)	# {2, 4, 5, 7, 8}
```
`list`와 `tuple`은 순서가 존재하기 때문에 인덱싱을 통해 자료형의 값을 조회할 수 있다.

하지만 `dictionary`와 `set`은 순서를 고려하지 않기 때문에 인덱싱을 활용할 수 없다.


### 전역 변수
```python
# 전역변수 사용법 1
def sub(a):
    # 함수 외부에서 선언했던 g 변수
    # 변수는 global 키워드가 필요하다.
    global g
    return g - a

g = 10
print(sub(3))       # 7


# 전역변수 사용법 2
def func():
    # 외부에서 선언했던 리스트
    # global 없이 접근 가능
    arr.append(5)

arr = [1, 2, 3, 4]
func()
print(arr)          # [1, 2, 3, 4, 5]
```


## 입출력

### 표준 입력
```python
# input() : 한 줄의 문자열을 입력 받음
s = input()		# string
n = int(input())	# integer

# split(문자) : 인자로 주어진 문자를 기준으로 구분함
s1 = input().split()	# 띄어쓰기를 기준으로
s2 = input().split(',')	# 콤마(,)를 기준으로

# map() : 리스트 모든 원소에 특정한 함수를 적용
s = input().split()	# ex. 1 2 3 4
print(s)		# ['1', '2', '3', '4']
n = map(int, s)		# s(리스트)에 모두 int 함수 적용
n = list(n)
print(n)		# [1, 2, 3, 4]

### 종합 ###
# 1. 공백으로 구분된 값을 입력받아 정수로 변환 후 리스트로 저장
arr = list(map(int, input().split()))

# 2. 각 변수에 정수를 입력 받을 때(ex. 3개)
a, b, c = list(map(int, input().split()))
print(a, b, c)
```

코딩 테스트에서 자주 자용하는 `입력 예제`는 다음과 같다.

```python
# 다량의 입력을 받는 경우(feat. 코딩테스트)

import sys

# 문자열 입력 받기(엔터 포함)
str = sys.stdin.readline()

# 문자열 입력 받기(엔터 제외)
str = sys.stdin.readline().rstrip()
```

### 표준 출력

```python
# 개행 포함
print("Hello")			# Hello
print("World")			# World

# 개행 제외
print("Hello", end = " ")
print("World", end = " ")	# Hello Wrold

# 변수 출력 1
num = 10
print("num : " + num)		# 에러 발생
print("num : " + str(num))	# num : 10

# 변수 출력 2 (f-string) (Python 3.6 이상)
num = 20
print(f"num : {num}")		# num : 20
```


## 조건문

파이썬의 코드 블록은 들여쓰기(`Indent`)로 구분한다. 이는 조건문, 반복문, 함수 등 모든 경우에 해당하며,

들여쓰기 기준은 4칸으로 한다.

```python
# 조건문 기본

a = 10

if a > 10 :
    print("a > 10")

if a == 10 :		# 또는 elif a == 10 :
    print("a == 10")

if a < 10 :		# 또는 else :
    print("a < 10")
    

# 조건문 안의 조건문
a = 6

if a % 2 == 0:
    if a % 3 == 0:
        print("2와 3의 공배수")
        
# 2개의 조건
a = 20

if a >= 10 and a < 30 :
    print("10 이상 30 미만")
```

```python
num = 100

# 조건문의 간소화 1
if num > 80: print("Pass")
else: print("Fail")

# 조건문의 간소화 2
# int a = (num == 100) ? num : 0    # C언어 스타일
a = num if num == 100 else 0    
print(a)	# 100

# 조건문의 간소화 3
if 80 <= num <= 100:
    print("80 이상 100 이하")
else:
    print("그 외")
```


## 반복문

```python
# 1 ~ 10까지 더하는 코드

# while문
num = 1
sum = 0

while num <= 10:
    sum += num
    num += 1

print(sum)


# for문 1
sum = 0

for i in range(1, 11):
    sum += i
print(sum)


# for문 2 (만약 리스트가 존재한다면)
sum = 0
arr = [i for i in range(1, 11)]

for i in arr:
    sum += i

print(sum)


# Break & Continue
sum = 0
for i in range(1, 9999):
    if i % 2 == 0:
        continue    # 짝수는 스킵

    if i > 10:      # 10 초과일 경우 중단
        break
        
    sum += i
    
print(sum)
```

## 함수

```python
# 더하기 함수
def add(a, b):
    return a + b

print(add(1, 3))    # 4

# 함수의 파라메터 지정
print(add(b = 3, a = 1))    # 4


# 다중 리턴 함수
def calc(a, b):
    add = a + b;
    sub = a - b;
    mul = a * b;
    div = a / b;
    return add, sub, mul, div

a, s, m, d = calc(10, 4)
print(a, s, m, d)   # 14 6 40 2.5
```

## 람다식
함수를 객체 형식으로 표현한 것

```python
# 람다 표현식 (더하기 함수)
add = (lambda a, b: a + b)(3, 5)
print(add)      # 8


############
# 자주 쓰는 람다 형식(정렬)
arr = [("사과", 1000), ("복숭아", 2000), ("토마토", 500)]

# 튜플의 참조(예시)
tup = ("AAA", 100)
print(tup[0], tup[1])   # AAA 100

# 람다 구현 (가격 기준 정렬)
print(sorted(arr, key=lambda x: x[1]))


############
# 자주 쓰는 람다 형식(리스트 활용)
lst1 = [1, 2, 3, 4,  5]
lst2 = [6, 7, 8, 9, 10]

lst = list(map(lambda a, b: a + b, lst1, lst2))
print(lst)
```
