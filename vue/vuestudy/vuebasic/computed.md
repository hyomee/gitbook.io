# Computed

Computed 속성은 Vue.js에서 데이터를 계산하고, 이를 템플릿에 바인딩할 수 있는 속성으로 기존 데이터를 가공하여 새로운 데이터를 만들어내는데 사용되는 것으로다음과 같은 특징을 가집진다

* Computed 속성은 기존 데이터를 가공하여 새로운 데이터를 만들어 낸다.
* Computed 속성은 캐싱되어 있어, 의존하는 데이터가 변경되지 않는 한 계산을 다시 하지 않으므로 성 요소의 성능이 훨씬 더 향상된다.
* Computed 속성은 템플릿에서 사용할 수 있습니다.
* Computed 속성은데이터에 대한 의존성을 가지므로기본적으로 getter 함수를 가지고 있다

&#x20;참고 : [Computed 속성 vue3 ](https://ko.vuejs.org/guide/essentials/computed.html)

Vue.observable은 주어진 객체의 반응형 인스턴스를 반환하는 함수로  데이터 함수에서 반환된 개체를 사용하여 개체를 반응하게 만드는데 내부적으로 사용 한다. 즉  데이터가변경되면 결과 개체는 렌더링 함수 및 Computed 속성 내에서 직접 사용할 수 있으며 필요한 수정을 트리거합니다.

<figure><img src="../../../.gitbook/assets/image (4).png" alt="" width="563"><figcaption></figcaption></figure>

* getter 를 통해 종속성을 추적 한다.
* setter 를 통해 변경을 알리게 된다
* Watcher 는 리렌더링을 위해 트리거한다.

