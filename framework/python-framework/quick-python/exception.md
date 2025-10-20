---
description: 예외 처리
---

# Exception

## 1. 예외 처리

파이썬에서 예외 처리는 프로그램 실행 중 발생할 수 있는 오류를 처리하여 프로그램이 중단되지 않고 계속 실행될 수 있도록    하는 기능입니다.

* 예외 처리: try, except, else, finally 키워드 사용
  * 예외 처리는 프로그앰의 안정성을 높이는 데 필수
  * try, except, else, finally를 적절히 조합하여 예외 처리
  * 필요에 따라 사용자 정의 예외를 만들어 사용할 수 있음
* 기본 구조:

<pre class="language-python"><code class="lang-python"><strong>try:
</strong>  # 예외가 발생할 가능성이 있는 코드
except 예외_타입 as 변수:
  # 예외가 발생했을 때 실행되는 코드
  # 예외 타입을 명시하지 않으면 모든 예외를 처리
else:
  # 예외가 발생하지 않았을 때 실행되는 코드
finally:
  # 예외 발생 여부와 상관없이 항상 실행되는 코드
  # 주로 자원 정리에 사용
</code></pre>

## 2. 예제 코드

```python
### 1. 기본 예외 처리
# 0으로 나누는 연산을 시도하여 ZeroDivisionError 예외를 발생시킴
try:
  result = 10 / 0  # 0으로 나누기 시도
except ZeroDivisionError as e:
  # ZeroDivisionError 예외가 발생했을 때 실행되는 코드
  print(f"예외 발생: {e}")  # 예외 메시지를 출력 : 예외 발생: division by zero

### 2. 여러 예외 처리
# 여러 종류의 예외를 처리하는 예제
try:
  value = int("abc")  # 문자열을 정수로 변환 시도 (ValueError 발생)
except ValueError:
  # ValueError 예외가 발생했을 때 실행되는 코드
  print("값 변환 오류 발생")  # 출력 : 값 변환 오류 발생
except ZeroDivisionError:
  # ZeroDivisionError 예외가 발생했을 때 실행되는 코드
  print("0으로 나눌 수 없습니다")

### 3. else와 finally 사용
# 예외 처리에서 else와 finally 블록을 사용하는 예제
try:
  result = 10 / 2  # 정상적인 나누기 연산
except ZeroDivisionError:
  # ZeroDivisionError 예외가 발생했을 때 실행되는 코드
  print("0으로 나눌 수 없습니다")
else:
  # 예외가 발생하지 않았을 때 실행되는 코드
  print(f"결과: {result}")  # 연산 결과 출력
finally:
  # 예외 발생 여부와 상관없이 항상 실행되는 코드
  print("프로그램 종료")  # 종료 메시지 출력

## 사용자 정의 예외
# 사용자 정의 예외 클래스를 정의
class CustomError(Exception):
  pass  # 사용자 정의 예외 클래스는 Exception을 상속받아 정의

# 사용자 정의 예외를 발생시키는 예제
try:
  raise CustomError("사용자 정의 예외 발생")  # CustomError 예외를 강제로 발생
except CustomError as e:
  # CustomError 예외가 발생했을 때 실행되는 코드
  print(e)  # 예외 메시지 출력

### 사용자 정의 예외와 raise
# 사용자 정의 예외를 발생시키고 처리하는 또 다른 예제
try:
  raise CustomError("사용자 정의 예외 발생")  # CustomError 예외를 강제로 발생
except CustomError as e:
  # CustomError 예외가 발생했을 때 실행되는 코드
  print(f"예외 처리: {e}")  # 예외 메시지 출력

print("*")  # 구분을 위한 출력

### 예외 전파
# 함수 내부에서 발생한 예외를 외부로 전파하는 예제
def divide(a, b):
  # 나누기 연산을 수행하며, b가 0일 경우 ZeroDivisionError 예외를 발생
  if b == 0:
    raise ZeroDivisionError("0으로 나눌 수 없습니다")  # 예외 강제 발생
  return a / b  # 나누기 결과 반환

def func():
  # divide 함수에서 발생한 예외를 처리
  try:
    divide(10, 0)  # b가 0이므로 ZeroDivisionError 발생
  except ZeroDivisionError as e:
    # ZeroDivisionError 예외가 발생했을 때 실행되는 코드
    print(f"예외 전파: {e}")  # 예외 메시지 출력

func()  # func 함수 호출
```

예외 발생: division by zero\
값 변환 오류 발생\
결과: 5.0\
프로그램 종료\
사용자 정의 예외 발생\
예외 처리: 사용자 정의 예외 발생\
\*\
예외 전파: 0으로 나눌 수 없습니다
