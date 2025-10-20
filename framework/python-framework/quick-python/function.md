# Function

## 1. 기본

* 수학에서의 함수(y=f(x))가 x에 대응하는 y를 갖는 것처럼, 프로그램의 함수도 동일한 입력값을 넣으면 그 일력값에 대응하는 결과가 있기 때문이다.
* 프로그램에서 함수도 동일한 코드를 반복해서 작성하지 않기 위해서 필요
*   함수 정의:

    * **def + 함수명(매개변수, 매개변수)**
    * **입력갑 전달**: 매개변수 이름을 명시적으로 작성을 하면
      * 코드를 이해하기 쉬워지며
      * 매개변수 이름을 넣으면 입력값의 순서를 지키지 않아도 된다.
    * **매개변수초깃값**: ( 매개변수\_01 = 값\_01, 매개변수\_02 = 값\_02)
      * 매개변수에 대해 입력값을 넣지 않았을 떄 초깃갑 적용
      *   주의사항:

          * 초깃값을 지정한 매개변수는 초깃값을 지정하지 않은 매개변수 다음에 선언되어야 함
          * 함수를 호출할 떄는 초깃값이 없는 매개변수에 입력값을 먼저 할당해야 함



    <figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption><p>파이썬 함수</p></figcaption></figure>

```python
def basic_operations(value1=1, value2=1, operation: str = 'add'):
    if operation == 'add':
        return value1 + value2
    elif operation == 'subtract':
        return value1 - value2
    elif operation == 'multiply':
        return value1 * value2
    elif operation == 'divide':
        if value2 != 0:
            return value1 / value2
        else:
            return "Error: 0으로 나누는 것은 허용되지 않습니다."
    else:
        return "Error: 지원되지 않는 작업."

# 일반적입 함수 호춯
print(basic_operations(10, 5, 'add'))        # 15

# 명시적으로 매개변수 명을 지정하여 호출하며 순서에 관계없이 호출
print(basic_operations(value1=10, value2=5, operation='add'))  # 15
print(basic_operations(value2=4, value1=10, operation='subtract'))  # 6

# 명시적으로 매개변수 명을 지정하는 것과 지정허지 않는 경우 매개변수 지정하지 않는 것아 앞으로 
print(basic_operations(10, value2=4, operation='subtract')) # 6

# 명시적으로 매개변수 명을 지정하는 경우 매개변수가 없는 경우 
# 함수 선언에 지정된 기본값을 사용
print(basic_operations(value1=10, value2=4))  # 14  
print(basic_operations(value1=10, operation='subtract'))  # 9
```

## 2. 함수 패킹/언패킹

* 디셔너리 패킹: 함수의 입력값을 딕셔너리 자료형으로 만드는 작업
  * 함수정의: \*\*변수명
  * 함수호출: 매개변수=값
* 딕션너리 언패킹: 딕셔너리 데이터를 함수의 매개변수와 값으로 바꾸는 작업
  * 함수정의: 매개변수=값
  * 함수호출: \*\*변수명

```python
# 함수 패킹 선언
def fn_packing(*args, **kwargs):
    print(f"args= {args}, kwargs= {kwargs}")

# 함수 패킹 선언
def fn_packing01(*args, **kwargs):
    print("fn_packing01 :=======")
    print(f"args= {args}, kwargs= {kwargs}")
    fn_unpacking(**kwargs)

# 함수 언패킹 선언
def fn_unpacking(a, b):
    print(f"a= {a}, b= {b}")

# kwargs 지정 하지 않음
fn_packing(1, 2, 3)  # args= (1, 2, 3), kwargs= {}

# kwargs 지정은 매개변수=값 으로 지정
fn_packing(1, 2, 3, a=4)  # args= (1, 2, 3), kwargs= {'a': 4}

fn_packing01(1, 2, 3, a=4, b=5)  

# 결과
# fn_packing01 :=======
# args= (1, 2, 3), kwargs= {'a': 4, 'b': 5}
# a= 4, b= 5
```

## 3. 변수에 함수 할당

* 변수에 함수를 선언하고 사용하는 방법
* 순수함수: 함수형 프로그래밍의 핵심 개념으로, 동일한 입력에 항상 동일한 출력을 내고, 외부 상태를 변경하지 않는(부수효과가 없는) 함수

```python
# 데이터의 합을 계산하는 함수
# 순수함수: 동일한 입력에 대해 항상 동일한 출력을 반환하며, 외부 상태를 변경하지 않음
def sum_data(data):
  return sum(data)

# 데이터의 평균을 계산하는 함수
# 순수함수: 동일한 입력에 대해 항상 동일한 출력을 반환하며, 외부 상태를 변경하지 않음
def avg_data(data):
  return sum(data) / len(data)

# 데이터를 처리하는 함수
# func 매개변수로 전달된 함수를 호출하여 데이터를 처리하고 결과를 출력
# 이 함수는 부수효과를 가짐: 결과를 출력(print)하는 동작이 외부 상태(콘솔 출력)에 영향을 미침
def process_data(data, func):
  result = func(data)  # 전달된 함수(func)를 호출하여 결과를 계산
  print(f"결과: {result}")  # 결과를 출력 (부수효과)

# 데이터 리스트
# 이미 정의된 data 변수를 사용
# data = [1, 2, 3, 4, 5]

# 데이터의 합을 계산하고 결과를 출력
process_data(data, sum_data)  # 결과: 15

# 데이터의 평균을 계산하고 결과를 출력
process_data(data, avg_data)  # 결과: 3.0
```

