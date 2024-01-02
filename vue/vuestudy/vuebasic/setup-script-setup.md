# setup() 와 script setup 차이점

Composition API 를 기반으로 Vue 3 프로젝트를 진행하는 방법에는 아래의 두 가지 방식이 있습니다.

#### 1. script setup

\<script setup>은 싱글 파일 컴포넌트(SFC) 내에서 컴포지션 API를 더 쉽게 읽거나 사용하기 위한 컴파일 타임 문법으로 SFC에서 컴포지션 API를 사용하는 경우, 권장되는 문법입니다. \<script> 문법에 비해 다음과 이점이 있습니다.

* 더 적은 상용구로 더 간결한 코드
* 순수 TypeScript를 사용하여 props 및 내보낼(emit) 이벤트를 선언하는 기능
* 더 나은 런타임 성능(템플릿은 중간 프락시 없이 동일한 범위의 렌더 함수로 컴파일됨)
* 더 나은 IDE 타입 추론 성능(언어 서버가 코드에서 타입을 추출하는 작업 감소)
* 참고 : vue 공식 [\<script setup>](https://ko.vuejs.org/api/sfc-script-setup.html#using-components)

```javascript
<script setup>
import { ref , capitalize } from './helpers'
import MyComponent from './MyComponent.vue'
// 변수
const msg = '안녕!'
const count = ref(0)
// 함수
function log() {
  console.log(msg)
}
</script>

<template>
  <!-- 템플릿에서 참조 변수직접 사용 -->
  <button @click="log">{{ msg }}</button>
  <!-- 템플릿에서 참조 함수 직접 사용 -->
  <div>{{ capitalize('hello') }}</div>
  <!-- 템플릿에서 ref는 언래핑되어 .value 없이 접근 가능 -->
  <button @click="count++">{{ count }}</button>
  <!-- 직접 접근 -->
  <MyComponent />
</template>
```

#### 2. setup()

다음과 같은 경우, 컴포넌트에서 컴포지션 API 사용을 위한 진입점 역할을 합니다

1. 빌드 과정 없이 컴포지션 API 사용.
2. 옵션 API 컴포넌트에서 컴포지션 API 기반 코드와 통합.
3. 참고 : vue 공식 [script()](https://ko.vuejs.org/api/composition-api-setup.html)

```javascript
<script>
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(0)

    // 템플릿 및 기타 옵션 API 훅에 노출
    return {
      count
    }
  },

  mounted() {
    console.log(this.count) // 0
  }
}
</script>

<template>
  <button @click="count++">{{ count }}</button>
</template>
```

공식 문서에서

1. Props에 접근 : setup 함수의 첫 번째 인자로   setup 함수 내부의 props를 전달 할 떄 사용 합니다. `props`는 반응형으로 props.xxx처럼 접근하는 것이 좋습니다.
2. Setup Context : setup 함수의 두번째 인자로   setup 함수 내부에서 attrs, slots, emit, expose을 사용 할 때 사용 합니다.

## 1. 차이점

1. \<script>는 컴포넌트를 처음 가져올 때 한 번만 실행되는데 \<script sctipt>는 내부 코드는 컴포넌트의 setup() 함수 내용으로 컴파일되므로 컴포넌트의 인스턴스가 생성될 때마다 실행됩니다.
2. \<script setup>은 내부에 선언된 모든 최상위 바인딩(변수, 함수 선언 및 import 포함)은 템플릿에서 직접 사용할 수 있습니다. 즉 변수, 함수, component를 변수 처럼 \<template> 에서 직접 사용 가능 합니다.



