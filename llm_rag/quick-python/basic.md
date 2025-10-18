# Basic

## 1. Python

* 컴파일 과정없이 줄 단위로 실행되는 인터프리터 언어
* 간결하고 직관적인
* 줄 단위로 실행되기 때문에 오류 코드까지 실행됨
* 학습 용이성, 개발 생산성으로 강력한 생태계 구성

## 2. 기본 타입

* 내장번수:
  * _\_\_name_\_\_: 현재 실행 중인 프로그램 이름
  * \_\_main\_\_: \_\_name\_\_에 \_\_main\_\_가 들어가 있음&#x20;

```python
def main():
    print("main 시행")

if __name__ = "__main__":
    print("main으로 실행되었습니다.")
    main()
```

### 2-1. 기본 타입

* 숫자형: 정수형, 실수향, 복소수형
  * 변수에 값을 할당하는 순간에 데이터 타입이 결정됨
  * 복소수: 복소수란 실수부와 허수부로 구성된 숫자입니다.
* 문자열: 작은 따옴표('), 큰 따옴표(")를 쌍으로 감싼 데이터
  * +: 문자열 연결
  * \*: 문자열 반복
  * 문자열 포메팅: 문자열안에 변수를 삽입하여 새로운 문자열을 만드는 방법
    * format 함수: 문자열 안에 중괄호({})를 사용하여 변수를 삽입할 위치를 지정
    * f-string(formatted-string): 문자열 앞에 f를 붙이고 중괄호({})안에 변수 삽입

```python
# -*- coding: utf-8 -*-
x = 10    # 정수
y = 3.14  # 실수
c = 3 + 4j  # 복소수
s = "Hello, World!" # 문자열 (알파벳:10, 특수문자:2, 스페이스:1 총:13)

print(type(x), x)  # <class 'int'>
print(type(y), y)  # <class 'float'>
print(type(c), c)  # <class 'complex'>
print(type(s), s, len(s))  # <class 'str'>

s1 = "Hello"
s2 = "World"
s3 = s1 + s2  # 문자열 연결
print(s1, type(s1), s3)  # Hello <class 'str'>
z = s1 * 3    # 문자열 반복
print("s1*3:", z)  # Hello <class 'str'>
print("="*10)

# 문자열 포매틸
print("x = %d, y = %.2f" % (x, y))  # x = 10, y = 3.14
print("x = {}, y = {}".format(x, y))  # x = 10, y = 3.14
print(f"x = {x}, y = {y}")  # x = 10, y = 3.14

# 문자열 메서드
print("문자열 메서드", "="*10)
print(s.lower())  # 문자열 소문자 변환
print(s.upper())  # 문자열 대문자 변환
print(s.split(",")) # 문자열 분할
p = '파이썬'
print(p.join(["Hello ", " World"]))  # 문자열 결합
```

### 2-2. 컬렉션

여러개의 데이터를 묶음으로 가지고 있는 데이터

#### 2-2-1. 리스트

여러 요소를 순서대로 저장하는 자료형

```python
userList = [1, '둘', 3, '넷', 5]  # 리스트
print(type(userList), userList)  # <class 'list'> [1, '둘', 3, '넷', 5]  # 리스트의 타입과 내용을 출력
print(userList[0], userList[-1])  # 1 5  # 첫 번째 요소와 마지막 요소를 출력
print(userList[1:3])    # ['둘', 3]  # 두 번째부터 세 번째 요소까지 슬라이싱
print(userList[1:])     # ['둘', 3, '넷', 5]  # 두 번째 요소부터 끝까지 슬라이싱
print(userList[:3])     # [1, '둘', 3]  # 처음부터 세 번째 요소까지 슬라이싱
print(userList[::2])    # [1, 3, 5]  # 처음부터 끝까지 2칸씩 건너뛰며 슬라이싱
print(userList[::-1])   # [5, '넷', 3, '둘', 1]  # 리스트를 역순으로 슬라이싱
print(userList[1:4:2])   # ['둘', '넷']  # 두 번째부터 네 번째 요소까지 2칸씩 건너뛰며 슬라이싱
print(userList[1:4:3])   # ['둘']  # 두 번째부터 네 번째 요소까지 3칸씩 건너뛰며 슬라이싱
# print(userList[1:4:0])   # 오류 발생  # 슬라이싱의 step 값이 0이면 ValueError 발생
print(userList[1:4:1])   # ['둘', 3, '넷']  # 두 번째부터 네 번째 요소까지 1칸씩 슬라이싱

# 추가 
userList.append(6)  # 리스트에 요소 추가
print(userList)  # [1, '둘', 3, '넷', 5, 6]  # 리스트에 요소 추가 후 출력
userList.insert(1, 2)  # 리스트의 두 번째 위치에 요소 추가
print(userList)  # [1, 2, '둘', 3, '넷', 5, 6]  # 리스트에 요소 추가 후 출력
userList.remove(2)  # 리스트에서 요소 제거
print(userList)  # [1, '둘', 3, '넷', 5, 6]  # 리스트에서 요소 제거 후 출력
userList.pop()  # 리스트의 마지막 요소 제거
print(userList)  # [1, '둘', 3, '넷', 5]  # 리스트에서 마지막 요소 제거 후 출력
userList.clear()  # 리스트의 모든 요소 제거
print(userList)  # []  # 리스트의 모든 요소 제거 후 출력
```

#### 2-2-2. 딕셔너리

* 키 중복이 없는 키:값 구조의 데이터
* 중괄호({})로 표시함

```python
animal_dic = { 'cat': '고양이', 'dog': '개', 'bird': '새'}  # 딕셔너리
print(type(animal_dic), animal_dic)  # <class 'dict'> {'cat': '고양이', 'dog': '개', 'bird': '새'}  # 딕셔너리의 타입과 내용을 출력
print("딕셔너리에서 'cat' 키의 값을 출력: ", animal_dic['cat'])  # 고양이  # 딕셔너리에서 'cat' 키의 값을 출력
print("딕셔너리에서 'dog' 키의 값을 출력: ", animal_dic.get('dog'))  # 개  # 딕셔너리에서 'dog' 키의 값을 출력
print("딕셔너리의 키를 출력: ", animal_dic.keys())  # dict_keys(['cat', 'dog', 'bird'])  # 딕셔너리의 키를 출력
print("딕셔너리의 값을 출력: ", animal_dic.values())  # dict_values(['고양이', '개', '새'])  # 딕셔너리의 값을 출력
print("딕셔너리의 키와 값을 출력: ", animal_dic.items())  # dict_items([('cat', '고양이'), ('dog', '개'), ('bird', '새')])  # 딕셔너리의 키와 값을 출력
print("딕셔너리에서 'cat' 키의 값을 출력: ", animal_dic.get('cat', '없음'))  # 고양이  # 딕셔너리에서 'cat' 키의 값을 출력
print("딕셔너리에서 'lion' 키의 값을 출력:", animal_dic.get('lion', '없음'))  # 없음  # 딕셔너리에서 'lion' 키의 값을 출력
print("*"*20)

# 값 추가 
animal_dic['lion'] = '사자'  # 딕셔너리에 'lion' 키와 '사자' 값을 추가
print("딕셔너리에 'lion' 키와 '사자' 값을 추가:", animal_dic)  # {'cat': '고양이', 'dog': '개', 'bird': '새', 'lion': '사자'}  # 딕셔너리에 값 추가 후 출력

# 값 수정
animal_dic['cat'] = '고양이 수정'  # 딕셔너리의 'cat' 키의 값을 수정
print("딕셔너리의 'cat' 키의 값을 수정: ", animal_dic)  # {'cat': '고양이 수정', 'dog': '개', 'bird': '새', 'lion': '사자'}  # 딕셔너리의 값 수정 후 출력

# 값 삭제
del animal_dic['cat']  # 딕셔너리에서 'cat' 키와 그 값을 삭제
print("딕셔너리에서 'cat' 키와 그 값을 삭제: ", animal_dic)  # {'dog': '개', 'bird': '새', 'lion': '사자'}  # 딕셔너리에서 값 삭제 후 출력
print("딕셔너리에서 'dog' 키와 그 값을 삭제하고 출력: ", animal_dic.pop('dog'))  # 개  # 딕셔너리에서 'dog' 키와 그 값을 삭제하고 출력
print("딕셔너리에서 값 삭제 후 출력: ", animal_dic)  # {'bird': '새', 'lion': '사자'}  # 딕셔너리에서 값 삭제 후 출력

print("="*10)
# 딕셔너리 list로 변환
animals = animal_dic.values()  # 딕셔너리의 값들만 가져옴
animals_list = list(animals)  # 딕셔너리의 값들을 리스트로 변환
print("딕셔너리의 값들을 리스트로 변환: ", animals_list)  # ['새', '사자']  # 딕셔너리의 값들을 리스트로 변환 후 출력

# 딕셔너리 키 list로 변환
animals = animal_dic.keys()  # 딕셔너리의 키들만 가져옴
animals_key_list = list(animals)  # 딕셔너리의 키들을 리스트로 변환
print("딕셔너리의 키들을 리스트로 변환: ", animals_key_list)  # ['bird', 'lion']  # 딕셔너리의 키들을 리스트로 변환 후 출력

print("딕셔너리 중복", "="*10)
# 딕셔너리 중복 
animal_dic2 = { 'cat': '고양이', 'dog': '개', 'bird': '새', 'cat': '고양이2'}  # 딕셔너리 중복
print(type(animal_dic2), animal_dic2)  # <class 'dict'> {'cat': '고양이2', 'dog': '개', 'bird': '새'}  # 딕셔너리의 타입과 내용을 출력
```

#### 2-2-3. Set

* 집합의 정의가 중복을 허용하지 않고 순서가 없는 요소들의 집합
* 중복된 요소는 하나만 남고 나머지는 삭제됨
* 집합의 요소는 변경 가능하지만, 집합 자체는 변경 불가능함
* 집합은 set(\[]), {} 로 표시

```python
set_data = {1, 2, 3}  # 집합
print(type(set_data), set_data)  # <class 'set'> {1, 2, 3, 4, 5}  # 집합의 타입과 내용을 출력
set_data2 = {3, 4, 5, 1, 3}  # 집합 중복
print("중복데이터 제거:" , set_data2)  # <class 'set'> {1, 2, 3, 4, 5}  # 집합의 타입과 내용을 출력
set_data3 = {3,2,1}

print("set_data == set_data3:", set_data == set_data3)  # True  # 집합의 내용이 같으면 True

print("합집합(set_data | set_data2):", set_data | set_data2)  # {1, 2, 3, 4, 5}  # 집합의 합집합
print("교집합(set_data & set_data2):", set_data & set_data2)  # {1, 3}  # 집합의 교집합
print("차집합(set_data - set_data2):", set_data - set_data2)  # {2}  # 집합의 차집합

```

#### 2-2-4. 튜플

* 리스트와 같은 시퀀스 자료형
* 괄호(())로 묶어서 사용
* 순서대로 저장 되고 중복을 허용하며 인텍스로 요소애 접근
* 리스트와 치이점은 튜플은 한번 만들어진 후 변경할 수 없음(**불변성**)

```python
tuple_data = (1, 2, 3, 3)  # 튜플
print(type(tuple_data), tuple_data)  # <class 'tuple'> (1, 2, 3)  # 튜플의 타입과 내용을 출력
print("튜플의 첫 번째 요소:", tuple_data[0])  # 1  # 튜플의 첫 번째 요소를 출력
print("튜플의 마지막 요소:", tuple_data[-1])  # 3  # 튜플의 마지막 요소를 출력
print("튜플의 슬라이싱:", tuple_data[1:3])  # (2, 3)  # 튜플의 슬라이싱
print("튜플의 슬라이싱:", tuple_data[1:])  # (2, 3, 3)  # 튜플의 슬라이싱
print("튜플의 슬라이싱:", tuple_data[:3])  # (1, 2, 3)  # 튜플의 슬라이싱
print("튜플의 슬라이싱:", tuple_data[::2])  # (1, 3)  # 튜플의 슬라이싱
print("튜플의 슬라이싱:", tuple_data[::-1])  # (3, 3, 2, 1)  # 튜플의 슬라이싱
print("튜플의 슬라이싱:", tuple_data[1:4:2])  # (2, 3)  # 튜플의 슬라이싱
print("튜플의 슬라이싱:", tuple_data[1:4:3])  # (2,)  # 튜플의 슬라이싱

# tuple_data[4] = 4 # TypeError: 'tuple' object does not support item assignment
```

#### 2-2-5. 패킹/언패킹

* 패킹: 여러개의 요소를 하나의 변수애 할당
* 언패킹: 하나의 변수에 들어 있는 요소를 여러개의 변수에 나누어 분배

```python
numbers = 1,2,3,4,5  # 튜플
print(type(numbers), numbers)  # <class 'tuple'> (1, 2, 3, 4, 5)  # 튜플의 타입과 내용을 출력

a, b, c, d, e = numbers  # 튜플 언패킹
print(a, b, c, d, e)  # 1 2 3 4 5  # 튜플 언패킹 후 출력

a, b, *c = numbers  # 튜플 언패킹
print(a, b, c)  # 1 2 [3, 4, 5]  # 튜플 언패킹 후 출력

def calculate(a, b):
    return a + b, a - b, a * b, a / b  # 덧셈, 뺄셈, 곱셈, 나눗셈을 반환하는 함수
result = calculate(10, 5)  # 함수 호출
print(result)  # (15, 5, 50, 2.0)  # 함수의 반환값을 출력

add, sub, mul, div = calculate(10, 5)
print(add, sub, mul, div)  # 15 5 50 2.0  # 언패킹 후 출력
```

## 3. 컬렉션안에 컬렉션

고객정보는 고객명,나이, 주소(우퍈번호, 상위주소, 하위주소)로 되어 있는 정로로 구성이 되는데 이때 딕셔너리안에 딕서너리 또는 리스트로 구성이 된다.

```python
from pprint import pprint

hong = {'name': '홍길동', 
        'age': 20, 
        'address': {'post':'234-123', 
                    'address':'서울시 송파구', 
                    'detail':'ㅇㅇAPT'}}  # 딕셔너리

minho = {'name': '민호',
         'age': 25, 
         'address': {'post':'123-456', 
                     'address':'서울시 강남구', 
                     'detail':'ㅇㅇ빌딩'}}  # 딕셔너리
pprint(hong)
pprint(minho)

# 결과
{'address': {'address': '서울시 송파구', 'detail': 'ㅇㅇAPT', 'post': '234-123'},
 'age': 20,
 'name': '홍길동'}
{'address': {'address': '서울시 강남구', 'detail': 'ㅇㅇ빌딩', 'post': '123-456'},
 'age': 25,
 'name': '민호'}
```

### 3-1. enumerate 함수

컬렉션 데이터를 순서대로 인텍스를 반환

* 리스트에서 데이터를 꺼내 올떄 위지 정보나 순번이 필요한 경우

```python
customers = [hong, minho]  # 리스트
for idx, element in enumerate(customers):
    print(f"idx: {idx}")  # 인덱스와 요소를 출력
    pprint(element)  # 요소를 예쁘게 출력
    print("="*10)
    print(f"이름: {element['name']}, 나이: {element['age']}, 주소: {element['address']['address']}")  # 이름, 나이, 주소를 출력
    print("="*10)
    print(f"우편번호: {element['address']['post']}, 상세주소: {element['address']['detail']}")  # 우편번호, 상세주소를 출력
    print("="*10)
    print()

print("start를 통해 시작 idx 지정 :","==="*10)
for idx, element in enumerate(customers, start=1):
    print(f"idx: {idx}")  # 인덱스와 요소를 출력
    print("="*10)
    print(f"이름: {element['name']}, 나이: {element['age']}")  # 이름, 나이, 주소를 출력
    print("="*10)
    print(f"우편번호: {element['address']['post']}, 상세주소: {element['address']['detail']}")  # 우편번호, 상세주소를 출력
    print("="*10)
    print()
```

```
# 결과
idx: 0
{'address': {'address': '서울시 송파구', 'detail': 'ㅇㅇAPT', 'post': '234-123'},
 'age': 20,
 'name': '홍길동'}
==========
이름: 홍길동, 나이: 20, 주소: 서울시 송파구
==========
우편번호: 234-123, 상세주소: ㅇㅇAPT
==========

idx: 1
{'address': {'address': '서울시 강남구', 'detail': 'ㅇㅇ빌딩', 'post': '123-456'},
 'age': 25,
 'name': '민호'}
==========
이름: 민호, 나이: 25, 주소: 서울시 강남구
==========
우편번호: 123-456, 상세주소: ㅇㅇ빌딩
==========

start 지정 원하는 위치에서 부터 : ==============================
idx: 2
==========
이름: 홍길동, 나이: 20
==========
우편번호: 234-123, 상세주소: ㅇㅇAPT
==========

idx: 3
==========
이름: 민호, 나이: 25
==========
우편번호: 123-456, 상세주소: ㅇㅇ빌딩
==========
```

### 3-2. resersed 함수

컬렉션 데이터를 역순으로 꺼내 사용

```python
customers_reversed = list(reversed(customers))  # 리스트를 역순으로 변환
print("리스트를 역순으로 변환:", customers_reversed)  # 리스트를 역순으로 변환 후 출력
for idx, element in enumerate(customers_reversed):
    print(f"idx: {idx}")  # 인덱스와 요소를 출력
    print("="*10)
    print(f"이름: {element['name']}, 나이: {element['age']}")  # 이름, 나이, 주소를 출력
    print("="*10)
    print(f"우편번호: {element['address']['post']}, 상세주소: {element['address']['detail']}")  # 우편번호, 상세주소를 출력
    print("="*10)
    print()
```

```
# 결과
리스트를 역순으로 변환: [{'name': '민호', 'age': 25, 'address': {'post': '123-456', 'address': '서울시 강남구', 'detail': 'ㅇㅇ빌딩'}}, {'name': '홍길동', 'age': 20, 'address': {'post': '234-123', 'address': '서울시 송파구', 'detail': 'ㅇㅇAPT'}}]
idx: 0
==========
이름: 민호, 나이: 25
==========
우편번호: 123-456, 상세주소: ㅇㅇ빌딩
==========

idx: 1
==========
이름: 홍길동, 나이: 20
==========
우편번호: 234-123, 상세주소: ㅇㅇAPT
==========
```

### 3-3. sorted 함수

* 원본데이터
* key: 어떤 값을 기준으로 정렬할지
* reverse: 역순 정렬

```python
def  sort_by_age(customer):
    return customer['age']  # 나이를 기준으로 정렬하는 함수
sorted_customers = sorted(customers, key=sort_by_age)  # 나이를 기준으로 정렬
print("나이를 기준으로 정렬:", sorted_customers)  # 나이를 기준으로 정렬 후 출력
for idx, element in enumerate(sorted_customers):
    print(f"idx: {idx}")  # 인덱스와 요소를 출력
    print("="*10)
    print(f"이름: {element['name']}, 나이: {element['age']}")  # 이름, 나이, 주소를 출력
    print("="*10)
    print()
    
sorted_reverse_customers = sorted(customers, key=sort_by_age, reverse=True)  # 나이를 기준으로 정렬
print("나이를 기준으로 정렬:", sorted_reverse_customers)  # 나이를 기준으로 정렬 후 출력
for idx, element in enumerate(sorted_reverse_customers):
    print(f"idx: {idx}")  # 인덱스와 요소를 출력
    print("="*10)
    print(f"이름: {element['name']}, 나이: {element['age']}")  # 이름, 나이, 주소를 출력
    print("="*10)
    print()
```

```
# 결과
나이를 기준으로 정렬: [{'name': '홍길동', 'age': 20, 'address': {'post': '234-123', 'address': '서울시 송파구', 'detail': 'ㅇㅇAPT'}}, {'name': '민호', 'age': 25, 'address': {'post': '123-456', 'address': '서울시 강남구', 'detail': 'ㅇㅇ빌딩'}}]
idx: 0
==========
이름: 홍길동, 나이: 20
==========

idx: 1
==========
이름: 민호, 나이: 25
==========

나이를 기준으로 정렬: [{'name': '민호', 'age': 25, 'address': {'post': '123-456', 'address': '서울시 강남구', 'detail': 'ㅇㅇ빌딩'}}, {'name': '홍길동', 'age': 20, 'address': {'post': '234-123', 'address': '서울시 송파구', 'detail': 'ㅇㅇAPT'}}]
idx: 0
==========
이름: 민호, 나이: 25
==========

idx: 1
==========
이름: 홍길동, 나이: 20
==========
```

## 4. 조건문 반복문

* for .. in 컬렉션
* 컴프리헨션:간결하고 직관적으로 컬렉션을 만드는 표현식

```python
# 0~9 까지 중에서 짝수 구함 
# range(10) : 으로 x를 0부터 9까지 생성하면서 if 조건을 만족하는 x를 리스트에 추가
even = [x for x in range(10) if x % 2 == 0]  # 리스트 내포를 사용하여 짝수를 구함
print("0~9 까지 중에서 짝수 구함:", even)  # 짝수를 구한 리스트를 출력 
# 0~9 까지 중에서 짝수 구함: [0, 2, 4, 6, 8]

# 0~9 까지 중에서 홀수 구함
odd = [x for x in range(10) if x % 2 != 0]  # 리스트 내포를 사용하여 홀수를 구함
print("0~9 까지 중에서 홀수 구함:", odd)  # 홀수를 구한 리스트를 출력
# 0~9 까지 중에서 홀수 구함: [1, 3, 5, 7, 9]

# 0~9 까지 중에서 3의 배수 구함
three = [x for x in range(10) if x % 3 == 0]  # 리스트 내포를 사용하여 3의 배수를 구함
print("0~9 까지 중에서 3의 배수 구함:", three)  # 3의 배수를 구한 리스트를 출력
# 0~9 까지 중에서 3의 배수 구함: [0, 3, 6, 9]
```
