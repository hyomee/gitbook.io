# Class

## 1.  객체 지향

#### 클래스

```
- 클래스(Class) 
  클래스는 객체를 생성하기 위한 설계도 또는 틀입니다. 클래스는 속성(데이터)과 메서드(기능)를 정의하며, 
  이를 기반으로 객체를 생성할 수 있습니다.  
  예: `class Car:`

- 객체(Object) 
  객체는 클래스를 기반으로 생성된 실체입니다. 클래스에서 정의된 속성과 메서드를 가지며, 
  프로그램에서 실제로 사용되는 데이터입니다.  
  예: `my_car = Car()`

- 인스턴스(Instance)  
  인스턴스는 특정 클래스에서 생성된 객체를 의미합니다. 즉, 객체와 인스턴스는 거의 같은 의미로 사용되지만, 
  "인스턴스"는 특정 클래스와의 관계를 강조할 때 사용됩니다.  
  예: `my_car는 Car 클래스의 인스턴스입니다.`
```

## 2. Python 에서 클래스

#### 클랙스의 주요 요소

* **속성(Attributes)**: 객체가 가지는 데이터(변수)
* **메서드(Methods)**: 객체가 할 수 있는 동작(함수)
* **생성자(Constructor, `__init__` 메서드)**: 객체를 생성할 때 자동으로 호출되어 초기화를 담당

```python
# Car 클래스 정의
class Car:
  # 클래스 초기화 메서드 (생성자)
  # 객체가 생성될 때 호출되며, 브랜드, 모델, 연도를 초기화
  def __init__(self, brand, model, year):   
    self.brand = brand  # 자동차 브랜드 (예: 현대)
    self.model = model  # 자동차 모델 (예: 소나타)
    self.year = year    # 자동차 제조 연도 (예: 2022)

  # 엔진을 켜는 메서드
  # 호출 시 자동차의 브랜드와 모델명을 포함한 메시지를 반환
  def start_engine(self):
    return f"{self.brand} {self.model}의 엔진이 켜졌습니다."

  # 엔진을 끄는 메서드
  # 호출 시 자동차의 브랜드와 모델명을 포함한 메시지를 반환
  def stop_engine(self):
    return f"{self.brand} {self.model}의 엔진이 꺼졌습니다."

# Car 클래스의 인스턴스 생성
# Hyundai 브랜드의 Sonata 모델, 2022년형 자동차 객체 생성
my_car = Car("Hyundai", "Sonata", 2022)

# start_engine 메서드 호출
# 자동차의 엔진을 켜는 메시지를 출력
print(my_car.start_engine()) # Hyundai Sonata의 엔진이 켜졌습니다.

# stop_engine 메서드 호출
# 자동차의 엔진을 끄는 메시지를 출력
print(my_car.stop_engine()) # Hyundai Sonata의 엔진이 꺼졌습니다.
```
