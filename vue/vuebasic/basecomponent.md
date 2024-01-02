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
* **반응성(Reactivity)**: Vue는 JavaScript 상태(State) 변경을 추적하고, 변경이 발생하면 DOM을 효율적으로 업데이트하는 것을 자동으로 수행한다.
