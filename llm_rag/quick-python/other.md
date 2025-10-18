# Other

## 1. Decorator

* 함수나 메서드의 동작을 수정하거나 확장할 수 있는 강력한 도구
* 다른 함수를 인수로 받아 새로운 함수를 반환하는 함수
* 이를 통해 코드의 재사용성을 높이고, 중복을 줄이며, 가독성을 향상
* 데코레이터는 `@` 기호를 사용하여 함수나 메서드 위에 선언
* 사례: **로깅, 실행 시간 측정, 캐싱, 인증, 권한 확인** 등 다양한 상황에서 데코레이터가 활용
  1. **로깅**: 함수 호출 전후에 로그를 기록합니다.
  2. **인증**: 사용자가 인증되었는지 확인합니다.
  3. **캐싱**: 동일한 입력에 대해 결과를 캐싱하여 성능을 향상시킵니다.
  4. **타이밍**: 함수 실행 시간을 측정합니다.
* 주요특징:
  1. **코드 재사용성**: 공통된 기능(예: 로깅, 인증, 캐싱 등)을 여러 함수에 쉽게 적용
  2. **가독성 향상**: 함수의 동작을 수정하거나 확장하는 코드를 함수 본문 외부로 분리.
  3. **유연성**: 함수뿐만 아니라 클래스 메서드에도 적용.

```python
# decorator_function은 데코레이터로, 다른 함수의 동작을 수정하거나 확장할 수 있습니다.
def decorator_function(original_function):
  # wrapper_function은 원래 함수의 동작 전후에 추가 작업을 수행합니다.
  def wrapper_function(*args, **kwargs):
    print(f"{original_function.__name__} 함수가 호출되기 전입니다.")  # 원래 함수 호출 전 출력
    result = original_function(*args, **kwargs)  # 원래 함수 실행
    print(f"{original_function.__name__} 함수가 호출된 후입니다.")  # 원래 함수 호출 후 출력
    return result  # 원래 함수의 결과 반환
  return wrapper_function  # wrapper_function을 반환하여 데코레이터로 사용 가능

# display 함수에 decorator_function 데코레이터를 적용합니다.
@decorator_function
def display():
  print("display 함수가 실행됩니다.")  # display 함수의 본래 동작

# display 함수를 호출하면 데코레이터가 적용된 동작이 실행됩니다.
display()

### 결과
# display 함수가 호출되기 전입니다.
# display 함수가 실행됩니다.
# display 함수가 호출된 후입니다.
######
```

### 1-1. 데코레이터 체이닝

여러 데코레이터를 하나의 함수에 적용

```python
# 첫 번째 데코레이터 정의
def decorator_one(func):
  def wrapper(*args, **kwargs):
    print("데코레이터 1 실행")  # 데코레이터 1의 동작
    return func(*args, **kwargs)  # 원래 함수 호출
  return wrapper

# 두 번째 데코레이터 정의
def decorator_two(func):
  def wrapper(*args, **kwargs):
    print("데코레이터 2 실행")  # 데코레이터 2의 동작
    return func(*args, **kwargs)  # 원래 함수 호출
  return wrapper

# 데코레이터 체이닝 적용
@decorator_one  # decorator_one이 먼저 적용됨
@decorator_two  # decorator_two가 나중에 적용됨
def say_hello():
  print("안녕하세요!")  # 원래 함수의 동작

# 함수 호출
say_hello()

### 출력 결과
# 데코레이터 1 실행
# 데코레이터 2 실행
# 안녕하세요!
```

### 1-2. 클래스 데코레이터

클래스를 데코레이터로 사용할 수도 있습니다. 이 경우 `__call__` 메서드를 구현하여 함수처럼 동작하도록 만듦

```python
class DecoratorClass:
  def __init__(self, original_function):
    self.original_function = original_function

  def __call__(self, *args, **kwargs):
    print(f"{self.original_function.__name__} 함수가 호출되기 전입니다.")
    result = self.original_function(*args, **kwargs)
    print(f"{self.original_function.__name__} 함수가 호출된 후입니다.")
    return result

@DecoratorClass
def display():
  print("display 함수가 실행됩니다.")

display()


### 출력 결과
# display 함수가 호출되기 전입니다.
# display 함수가 실행됩니다.
# display 함수가 호출된 후입니다.

```

## 2. Annotation & Type Hint

* Python Annotation은 함수의 매개변수와 반환값에 대한 메타데이터를 제공하기 위해 사용
* 코드의 가독성을 높이고, 함수의 사용 방법을 명확히 하며, 정적 분석 도구나 IDE에서 유용하게 활용
* 주요 특징:
  * **가독성 향상**: 함수의 입력과 출력 타입을 명시적으로 나타낼 수 있음
  * **유연성**: 타입 힌트를 제공하지만, 강제하지 않으므로 동적 타입 언어의 특성을 유지
  * **도구 지원**: 정적 분석 도구, IDE, 문서 생성 도구에서 활용 가능
* 문법:
  * 매개변수의 타입 힌트는 `:` 뒤에 작성.
  * 반환값의 타입 힌트는 `->` 뒤에 작성하며, 함수 선언의 끝에 위치.
* 주의사항:
  * 타입 힌트는 권장사항일 뿐, Python은 이를 강제하지 않음.
  * 런타임에는 타입 힌트가 무시되며, 코드 실행에 영향을 미치지 않음.
  * 타입 힌트를 활용하려면 정적 분석 도구(예: `mypy`)를 사용하는 것이 좋음.

```python
# `a: int`는 `a` 매개변수가 정수형이어야 함을 나타냅니다.
# `b: int`는 `b` 매개변수가 정수형이어야 함을 나타냅니다.
# `-> int`는 함수가 정수형 값을 반환함을 나타냅니다.
def add_numbers(a: int, b: int) -> int:
  return a + b
```

### 2-1. 활용 예시

```python
# 1. **기본 타입 힌트** 
def greet(name: str) -> str:
  return f"Hello, {name}!" 

# 2. **컬렉션 타입 힌트**

from typing import List

def sum_list(numbers: List[int]) -> int:
  return sum(numbers)
 

# 3. **옵셔널 타입**
from typing import Optional

def get_length(s: Optional[str]) -> int:
  if s:
    return len(s)
  return 0
 

# 4. **커스텀 클래스 타입**

class Person:
  def __init__(self, name: str):
    self.name = name

def introduce(person: Person) -> str:
  return f"My name is {person.name}."
```
