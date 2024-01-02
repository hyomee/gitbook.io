# 확장 큐브 모델

애드리안 콕크로프트는 "마이크로 서비스 아키텍처는 경계 컨텍스트(Bounded Context)가 있는 느슨하게 결합된 엘리먼트(Element)로 구성된 서비스 지향 아키택처"라 정의 하였다. 어려운 이야기 입니다.

이 정의를 이해 하기 전에 규모 확장 모델을 이해해야 합니다.

<figure><img src="../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

확장큐브 모델은 로드 발란서 뒤에서 애플리케이션을 확보하는 방법에  모델로  x 축확장, y축 확장, z축 확장이 있습니다

## 1. X 축 확장

일반적인 모놀리식 애플리케이션의 확장 수단으로 로드 발란서 뒤에 있는 애플리케이션의 인스턴스를 N개 뛰워 놓는것입니다.

<figure><img src="../.gitbook/assets/image (47).png" alt="" width="563"><figcaption></figcaption></figure>

## 2. Z축 확장

X축 확장이 동일한 애플리케이션의 복제본으로 분산하는 확장이라면    Y축 확장은 동일한 애플리케이션의 복제본을 사용하지만 처리 데이터를 기준으로 그룹핑 하여 분산 하는 방법으로 데이터를 기준으로 분산 하는 방법입니다.

<figure><img src="../.gitbook/assets/image (165).png" alt="" width="563"><figcaption></figcaption></figure>

이 확장은 많은 데이터를 분산하여 처리 하기 위한 방법으로 배치 또는 데몬 애플리케이션에서도 응용되어 사용되고 있습니다.

## 3. Y축 확장

X, Z 축 확장이 애플리케이션의 요청에 의한 처리 기준으로 분산을 하는 방법이라면 Y축 확장은 기능을 분리 하여 분산 처리 하는 방법입니다. 즉 단일 애플리케이션을 기능 단위로 분리하여 다수의 애플리케이션을 만들어 확장 하고 각 기능별 애플리케이션은 X, Y 축 확장을 사용하여 분산 시키는 방법으로 Y축 확장이 마이크로 서비스의 기본 개념 입니다.

<figure><img src="../.gitbook/assets/image (166).png" alt="" width="563"><figcaption></figcaption></figure>

