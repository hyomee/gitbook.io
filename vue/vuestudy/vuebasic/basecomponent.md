# 기본 구성

## 1. SFC(Single File Component)

Vue에서 사용하는 하나의 파일을 의미 하며 다음과 같이 구성이 된다.

* template : 렌더링 되어 HTML 코드로 변환 되는 부분
* script : 자바스크립트 또는   타입스크립트  코드가 작성 되는 부분&#x20;
* style : CSS 코드로 scoped 속성이 작성 되어 있으면 해당 컴포넌트만 적용이 되고 없으면 전체 프로젝트에 영향을 준다.
* 커스텀 블럭 : 프로젝트별 요구 사항에 따라 `*.vue` 파일에 추가 커스텀 블록을 포함할 수 있습니다. ( 자세한 내용은 [SFC 커스텀 블록 통합 도구 섹션](https://ko.vuejs.org/guide/scaling-up/tooling.html#sfc-custom-block-integrations)을 참고 해주세요.)

```javascript
<template>
  <div class="example">{{ msg }}</div>
</template>

<script>
export default {
  data() {
    return {
      msg: '안녕 Vue!'
    }
  }
}
</script>

<style>
.example {
  color: red;
}
</style>

<custom1>
예를 들어 컴포넌트 설명서가 될 수 있습니다.
</custom1>
```

참고 : [공식 VUE SFC 설명서](https://ko.vuejs.org/api/sfc-spec.html#overview)

### 1-1. Vue 핵심 기능

* **선언적 렌더링(Declarative Rendering)**: Vue는 표준 HTML을 템플릿 문법으로 확장하여 JavaScript 상태(State)를 기반으로 화면에 출력될 HTML을 선언적(declaratively)으로 작성할 수 있다.
  * 즉, 변수를 선언하고 값을 넣으면 자동으로 DOM에 업데이트 되는 것
* **반응성(Reactivity)**: Vue는 JavaScript 상태(State) 변경을 추적하고, 변경이 발생하면 DOM을 효율적으로 업데이트하는 것을 자동으로 수행한다.

## 2. Vue.js 아키텍처 이해

Vue 앱의 아키텍처는 MVVM(Model-View-ViewModel) 패턴을 따르며, 이는 관심사를 분리하고 깨끗하고 체계적인 코드베이스로.작성하게 해준다.   이는 View, ViewModel 및 Model의 세 가지 주요 부분으로 구성된다.

<figure><img src="../../../.gitbook/assets/image (205).png" alt="" width="375"><figcaption><p>Vue.js 아키텍처 다이어그램</p></figcaption></figure>

### 2-1. ViewModel&#x20;

* ViewModel은 View와 Model 사이의 다리로 UI를 렌더링하고 업데이트하는 데 필요한 데이터와 로직을 관리하는 중개자 역할을 한다.
* &#x20;Vue.js에서 ViewModel은 Vue 생성자로 생성된 Vue 인스턴스를 사용하여 구현한다.&#x20;
* 인스턴스는 뷰의 동작을 제어하는 ​​데이터, 메서드, 계산된 속성 및 수명 주기 후크를 캡슐화 한다. 즉애플리케이션의 중앙 제어 지점 역할을 한다.

### 2-2. 모델&#x20;

* 모델은 애플리케이션의 데이터와 비즈니스 로직을 의미하며.  PI, 사용자 입력 또는 기타 소스에서 검색된 데이터가 포함될 수 있다.
* Vue.js에서 모델은 일반 JavaScript 객체이거나 ViewModel에 바인딩된 배열일 수 있으며 Model의 데이터가 변경될 때마다 ViewModel은 연결된 View를 자동으로 업데이트하여 반응적이고 동기화된 UI를 보장한다.

### 2-3. **작동 방식**&#x20;

* Vue.js는 가상 DOM(Document Object Model)을 활용하여 데이터가 변경될 때 UI를 효율적으로 업데이트 한다.
* Vue 인스턴스가 생성되면 템플릿을 분석하고 DOM의 가상 표현을 생성하고 이 가상 DOM은 실제 DOM과 ViewModel 사이의 중개자 역할을 하여 Vue.js가 효율적인 비교 및 ​​패치 알고리즘을 수행할 수 있도록 한다.
* ViewModel의 데이터가 변경되면 Vue.js는 이러한 변경 사항을 반영하는 데 필요한 최소한의 DOM 업데이트를 계산하여 최적의 성능을 보장한다.

### 2-4.**주요 기능 및 이점** : - 반응형 데이터 바인딩: Vue.js는 데이터가 변경될 때 UI를 자동으로 업데이트하는 반응형 시스템을 제공하여 수동 DOM 조작의 필요성을 최소화합니다. - 구성 요소 기반 아키텍처==> Vue.js는 재사용 가능하고 구성 가능한 구성 요소의 개발을 장려하여 코드 재사용 및 유지 관리를 촉진합니다. - Vue 라우터==> Vue Router는 클라이언트 측 라우팅을 지원하는 내장 라이브러리로, 여러 보기가 있는 단일 페이지 애플리케이션(SPA)을 생성할 수 있습니다. - Vuex==> Vuex는 Vue.js 애플리케이션을 위한 상태 관리 패턴이자 라이브러리입니다. 구성 요소 간의 공유 데이터를 관리하는 데 도움이 되므로 복잡한 애플리케이션 상태를 더 쉽게 처리할 수 있습니다.
