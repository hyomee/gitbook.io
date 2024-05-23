# GoF(Gang of Four)

디자인 패턴이란 특정 기능을 만들기 위한 클래스들의 연결한것으로 각 클래스의 관계 정리 한 것으로 C언어로 작성된 GoF(Gang-Of-Four) 패턴이 객체지향에 맞는디자인 패턴을 정리한 것이고 자바 J2EE 관련은 [**Core J2EE Patterns**](http://www.corej2eepatterns.com/)에 정리 되어 있습니다.

## 1. 디자인 패턴 이란

* 소프트웨어 설계에서 반복적으로 발생하는 문제들에 대한 해결책을 제공하는 일종의 베스트 프랙티스입니다.&#x20;
* 개발자들이 더 효율적이고 재사용 가능한 코드를 작성할 수 있도록 도와줍니다.&#x20;
*   디자인 패턴은 GoF(Gang of Four)의 23가지 패턴으로, 이들은 크게 생성(Creational), 구조(Structural), 행동(Behavioral) 패턴으로 분류됩니다.\
    \


    <figure><img src="../../.gitbook/assets/image (273).png" alt=""><figcaption></figcaption></figure>

## 2. 생성패턴 (Creational Patterns)

* 객체를 생성하는 방법에 중점을 둔 디자인 패턴으로 객체 생성에 관련된 로직을 캡슐화 하여 코드의 재사용성을 향상 시킨다&#x20;
* <mark style="color:purple;">**대표적인 패턴**</mark>&#x20;
  * **팩토리 패턴 (Factory Method)**: 팩토리 클래스를 통해서 객체 생성 로직을 캡술화하는 패턴 (인스턴스를 만드는 절차를 추상화하는 패턴:하위 클래스에게 인스턴스 작성)&#x20;
  * **추상 팩토리 (Abstract Factory)**: 여러 개의 관련된 팩토리 메서드를 함께 사용하여 부품을 조립하듯이 객체를 생성하는 패턴&#x20;
  * **빌더 (Builder)**: 객체 생성 과정을 단계별로 분리한 패턴 ( 선택적 매개변수를 가지지고 있는 객체를 생성할 때 유용)&#x20;
  * **프로토타입(Prototype):** 복사해서 인스턴스를 만드는 패턴 ( 기존 객체를 복제하여 새로운 객체를 생성 )&#x20;
  * **싱글톤 (Singleton Pattern)**: 클래스의 인스턴스가 한 개만 생성되도록 보장하는 패턴

## 3. 구조패턴 (Structural Patterns)

* 클래스나 객체를 조합하여 더 큰 구조를 만드는 디자인 패턴으로 코드의 재사용성을 높이고 객체 간의 관계를 재정의하여 유연한 프로그램 구조를 제공합니다.&#x20;
* <mark style="color:purple;">**대표적인 패턴**</mark>&#x20;
  * **퍼사드 패턴(Façade Pattern)**: 복잡한 서브시스템에 대해 하나의 통합된 인터페이스를 제공하여 시스템 구조를 단순하게 만드는 패턴&#x20;
  * **데코레이터 팩토리 (Decorator Pattern)**: 기존 객체의 기능을 동적으로 추가하거나 확장하는 방법을 제공하는 패턴 ( 객체를 래핑하여 새로운 기능을 추가 )&#x20;
  * **프록시 (Proxy Pattern)**: 객체의 접근을 제어하는 패턴 ( 프록시를 통해 객체의 생성, 소멸, 네트워크 통신 등을 관리 )&#x20;
  * **Adapter, Bridge, Composite, Flyweight**

## 4. 행동패턴 (Behavioral Patterns)

* 객체를 생성하는 방법에 중점을 둔 디자인 패턴으로객체 생성에 관련된 로직을 캡슐화 하여 코드의 재사용성을 향상 시킨다.
* <mark style="color:purple;">**대표적인 패턴**</mark>&#x20;
  * **커멘드 패턴 (Command Pattern)**: 인스턴스를 만드는 절차를 추상화하는 패턴 하위 클래스에서 인스턴스 작성&#x20;
  * **옵저버 팩토리 (Observer Pattern)**: 객체의 상태 변경을 다른 객체들에게 전파하는 패턴 ( Subject와 Observer 인터페이스를 활용하여 구현 ) 이벤트 기반 아키텍처에 활용되며, 객체 간의 결합도를 낮추어 독립적으로 동작할 수 있다.&#x20;
  * **Chain of Responsibility, Interpreter, Iterator, Mediator, Memento, State, Strategy, Template Method, Visitor**



{% file src="../../.gitbook/assets/JavaDesignPattern_20240228 (1).pdf" %}
