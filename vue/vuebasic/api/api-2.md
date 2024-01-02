# 컴포지션 API

## 1. 컴포지션 API&#x20;

<figure><img src="../../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

* 참고 : 이 문서는 ko.vuejs.org를  참고해서 작성 되었으며 [**script setup**](https://ko.vuejs.org/api/composition-api-setup.html)  [**setup**](https://ko.vuejs.org/api/composition-api-setup.html#basic-usage) 을 자세한 정보를 얻을 수 있다.

## 2. script setup

* \<script setup> \</script>로 작성
  * setup : 컴포지션 API를 의미하며,  컴파일 시 올바르게 동작할 수 있게 코드를 변환하도록 하는 힌트
  * \<script setup>에 import 되어 가져온 객체들과 선언된 최상위 변수 및 함수는 템플릿에서 직접 사용할 수 있다.
* 참고 : 주요 기능은 [vue 3 script setup](https://ko.vuejs.org/api/sfc-script-setup.html) 확인할 수 있다

### 2-1.  기본[​](https://ko.vuejs.org/api/sfc-script-setup.html#basic-syntax) <a href="#basic-syntax" id="basic-syntax"></a>

문법을 선택하려면 `<script>` 블록에 `setup` 속성을 추가해야 한다:

```html
<script setup>
    console.log('안녕, script setup!')
</script>
```

* 내부 코드는 컴포넌트의 `setup()` 함수 내용으로 컴파일된다.
* `<script>는` 컴포넌트를 처음 가져올 때 한 번만 실행되는데 `script setup>` 내부의 코드는 컴포넌트의 인스턴스가 생성될 때마다 실행된다**.**

#### 2-1-1. 템플릿에 직접 바인딩[​](https://ko.vuejs.org/api/sfc-script-setup.html#top-level-bindings-are-exposed-to-template) <a href="#top-level-bindings-are-exposed-to-template" id="top-level-bindings-are-exposed-to-template"></a>

내부에 선언된 모든 최상위 바인딩(변수, 함수 선언 및 `import` 포함)은 템플릿에서 직접 사용한다.

```html
<script setup>
  // 변수
  const msg = '안녕!'

  // 함수
  function log() {
    console.log(msg)
  }
</script>

<template>
  <button @click="log">{{ msg }}</button>
</template>
```

`methods` 옵션을 통해 노출하지 않고도 템플릿 표현식에서 `import`한 헬퍼 함수를 직접 사용한다.

```html
<script setup>
  import { capitalize } from './helpers'
</script>

<template>
  <div>{{ capitalize('hello') }}</div>
</template>
```

### 2-2. 반응형[​](https://ko.vuejs.org/api/sfc-script-setup.html#reactivity) <a href="#reactivity" id="reactivity"></a>

반응형 상태는 반응형 API를 사용하여 명시적으로 생성해야 하고 `ref함수를 사용한다.`

```html
<script setup>
  import { ref } from 'vue'
  const count = ref(0)
</script>

<template>
  <!-- 템플릿에서 ref는 언래핑되어 .value 없이 접근 가능 -->
  <button @click="count++">{{ count }}</button>
</template>
```

### 2-3. 컴포넌트  <a href="#using-components" id="using-components"></a>

PascalCase 컴포넌트 태그로 `<script setup>` 범위의 커스텀 컴포넌트 값은 태그 이름으로 사용한다.

```html
<script setup>
  import MyComponent from './MyComponent.vue'
</script>

<template>
  <MyComponent />
</template>
```

`MyComponent`는 변수처럼 참조된다고 생각하십시오. JSX를 사용한 적이 있는 경우, 멘탈 모델(mental model)과 유사합니다. kebab-case에 해당하는 `<my-component>`도 템플릿에서 작동하지만, 일관성을 위해 PascalCase 컴포넌트 태그를 강력히 권장합니다. 또한 네이티브 커스텀 엘리먼트와 구별하는 데 도움이 됩니다.

{% hint style="info" %}
컴포넌트 사용

kebab-case :  `<my-component>`도 템플릿

PascalCase : \<MyComponent />
{% endhint %}

#### 2-3-1.  동적 컴포넌트[​](https://ko.vuejs.org/api/sfc-script-setup.html#dynamic-components) <a href="#dynamic-components" id="dynamic-components"></a>

동적 컴포넌트를 사용할 때, 동적인 `:is` 바인딩을 사용하며 삼항 표현식에서 컴포넌트를 변수로 사용할 수 있다.

```html
<script setup>
  import Foo from './Foo.vue'
  import Bar from './Bar.vue'
</script>

<template>
  <component :is="Foo" />
  <component :is="someCondition ? Foo : Bar" />
</template>
```

#### 2-3-2. 재귀 컴포넌트[​](https://ko.vuejs.org/api/sfc-script-setup.html#recursive-components) <a href="#recursive-components" id="recursive-components"></a>

```javascript
import { FooBar as FooBarChild } from './components'
```

#### 2-3-3. 네임스페이스 컴포넌트[​](https://ko.vuejs.org/api/sfc-script-setup.html#namespaced-components) <a href="#namespaced-components" id="namespaced-components"></a>

단일 파일에서 여러 컴포넌트를 하는 경우 사용한다.

```html
<script setup>
  import * as Form from './form-components'
</script>

<template>
  <Form.Input>
    <Form.Label>레이블</Form.Label>
  </Form.Input>
</template>
```

### `2-4. defineProps()` & `defineEmits()`[​](https://ko.vuejs.org/api/sfc-script-setup.html#defineprops-defineemits) <a href="#defineprops-defineemits" id="defineprops-defineemits"></a>

`props` 및 `emits와 같은 기능으로` `defineProps` 및 `defineEmits` API를 사용한다.

<pre class="language-html"><code class="lang-html">&#x3C;script setup>
  const props = defineProps({
    foo: String
  })

<strong>  const emit = defineEmits(['change', 'delete'])
</strong>  // ... setup 코드
&#x3C;/script>
</code></pre>

* `defineProps` 및 `defineEmits`는 `<script setup>` 내에서만 사용할 수 있는 컴파일러 매크로`import`할 필요가 없으며, `<script setup>`이 처리될 때 컴파일된.
* `defineProps`는 `props` 옵션과  동일하고 `efineEmits`는 `emits` 옵션과 동일한  다.
* `defineProps` 및 `defineEmits`는 전달된 옵션을 기반으로 적절한 타입 추론을 제공한다..
* `defineProps` 및 `defineEmits`에 전달된 옵션은 setup 범위에서 선언된 로컬 변수는  호이스팅 되므로컴파일 에러가 발생  한다. 그러나 `import`한 바인딩은 모듈 범위에 있으므로 참조한다.

### 2-5. defineExpose()[​](https://ko.vuejs.org/api/sfc-script-setup.html#defineexpose) <a href="#defineexpose" id="defineexpose"></a>

`<script setup>`을 사용하는 컴포넌트는 기본적으로 닫혀 있으므로 `<script setup>` 내부에서 선언된 바인딩을 부모에서.접근을 할 수 없다. 부모가  접근하도록 하는 기능으로\템플릿 참조를 통해 이 컴포넌트의 인스턴스를 가져오면, 검색된 인스턴스는 `{ a: number, b: number }` 모양이 됩니다(참조는 일반 인스턴스와 마찬가지로 자동으로 언래핑됨).

```html
<script setup>
  import { ref } from 'vue'
  
  const a = 1
  const b = ref(2)
  
  defineExpose({
    a,
    b
  })
</script>
```

### 2-6. 일반 `<script>`와 함께 사용[​](https://ko.vuejs.org/api/sfc-script-setup.html#usage-alongside-normal-script) <a href="#usage-alongside-normal-script" id="usage-alongside-normal-script"></a>

`<script setup>`은 일반 `<script>`와 함께 사용할 수 있으며. 다음을 수행해야 하는 경우 일반 `<script>`가 필요할 수 있습니다:

* `<script setup>`에서 표현할 수 없는 옵션을 선언하는 경우. 예를 들어 `inheritAttrs` 또는 플러그인을 통해 활성화된 커스텀 옵션이 있는 경우 (3.3 이상에서 [`defineOptions`](https://ko.vuejs.org/api/sfc-script-setup.html#defineoptions)로 대체 가능).
* 명명된 `export`를 선언하는 경우.
* 사이드 이펙트를 실행하거나 한 번만 실행되어야 하는 객체를 만드는 경우.

```html
<script>
  // 일반 <script>, 모듈 범위에서 실행(한 번만)
  runSideEffectOnce()
  
  // 추가 옵션 선언
  export default {
    inheritAttrs: false,
    customOptions: {}
  }
</script>

<script setup>
// setup() 범위에서 실행(각 인스턴스에 대해)
</script>
```

다음은 사용하지 말아야 한다.

* `prop` 및 `emits`와 같이 `<script setup>`을 사용하여 이미 정의할 수 있는 옵션에 대해서는 별도의 `<script>` 섹션을 사용하지 마세요.
* `<script setup>` 내에서 생성된 변수는 컴포넌트 인스턴스에 프로퍼티로 추가되지 않으므로 옵션 API에서 액세스할 수 없습니다. 이런 식으로 API를 혼합하는 것은 강력히 권장하지 않습니다.

**위 사항  중 하나에 해당하는 경우 `<script setup>`을 사용하는 대신 명시적인 `setup()` 함수로 전환하는 것을 고려해야 한다.**

### 2-7. 최상위 `await`[​](https://ko.vuejs.org/api/sfc-script-setup.html#top-level-await) <a href="#top-level-await" id="top-level-await"></a>

최상위 `await`는 `<script setup>` 내에서 사용할 수결있다.&#x20;

```
<script setup>
const post = await fetch(`/api/post/1`).then((r) => r.json())
</script>
```

### 2-8.  제한사항[​](https://ko.vuejs.org/api/sfc-script-setup.html#restrictions) <a href="#restrictions" id="restrictions"></a>

* `<script setup>` 는 `src` 속성과 함께 사용할 수 없습니다.
* `<script setup>`은 In-DOM Root Component Template을 지원하지 않습니다.&#x20;

### 2-9. 예제

import를 사용해서 선언하면 바로 템플릿에 적용이 된다.

{% code title="App.vue" lineNumbers="true" %}
```html
<template>
  <MyComponent msg="Vue 3 학습" /> 
  <TemplateView />               
</template>

<script setup>
  import MyComponent from './views/MyComponent.vue';  
  import TemplateView from './views/TemplateView.vue';    
</script>

<style scoped>
</style> 
```
{% endcode %}

* 1\~5  line : 2 line 템플릿 정의&#x20;
  * MyComponent, TemplateView 컴포넌트를 선언함
* 8\~9 line : MyComponent, TemplateView 컴포넌트 import
  * script setup 를 사용하므로 바로 적용이 된다.

{% code title="MyComponent.vue" lineNumbers="true" %}
```javascript
<template>
  <h1>{{ propMsg }}</h1>

  <div class="card">
    <button type="button" @click="fnInc">count is {{ count }}</button>
    <p>
      Edit
      <code>components/FirstVite.vue</code> to test HMR
    </p>
  </div>
</template>

<script setup>
  import { ref, defineProps } from 'vue'

  // 1) Component 프로퍼티 선언 
  const props = defineProps({
    msg: String,
  })

  // 2) Component에서 사용할 변수 선언 
  const propMsg = ref(props.msg)
  const count = ref(0)

  // 3) Component에서 사용할 함수 선언 
  const fnInc = () => {
    count.value ++ ;
  }
 </script>

<style scoped> 
</style>
```
{% endcode %}

## 3. setup()

`setup()` 옵션 API 컴포넌트에서 컴포지션 API 기반 코드와 같이 사용한다.

### 3-1. 기본 사용법[​](https://ko.vuejs.org/api/composition-api-setup.html#basic-usage) <a href="#basic-usage" id="basic-usage"></a>

반응형 API를 사용하여 반응형 상태를 선언하고&#x20;

* `setup()`에서 객체를 반환
* &#x20;`setup`에서 반환된 refs는 템플릿에서 접근할 때 자동으로 얕은 언래핑되므로, 접근할 때 `.value`를 사용할 필요 없으며 `this`에서 접근한다.
* 객체를 _동기적으로_ 반환해야 한다.
  * `async setup()`을 사용할 수 있는 유일한 경우는 컴포넌트가 [Suspense](https://ko.vuejs.org/guide/built-ins/suspense.html) 컴포넌트의 자손인 경우이다.

```html
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

### 3-2. Props에 접근하기[​](https://ko.vuejs.org/api/composition-api-setup.html#accessing-props) <a href="#accessing-props" id="accessing-props"></a>

`setup` 함수의 첫 번째 인자는 `props이며 반응형으로`새 props가 전달되면 업데이트된다.

```
export default {
  props: {
    title: String
  },
  setup(props) {
    console.log(props.title)
  }
}
```

* `props` 객체를 분해할 경우, 분해 된 변수는 반응성을 잃게 되므로.항상 `props.xxx`처럼 접근하는 것이 다.
* Props를 분해해야 하거나, 반응성을 유지하면서 외부 함수에 props를 전달해야 하는 경우, toRefs() 또는 toRef() 유틸리티 API를 사용하여 구현할 수 있습니다.&#x20;

```javascript
import { toRefs, toRef } from 'vue'

export default {
  setup(props) {
    // refs 객체로 `props`를 변환한 후, 분해 할당
    const { title } = toRefs(props)
    // `title`은 `props.title`을 추적하는 ref 입니다.
    console.log(title.value)

    // 또는, 하나의 `props` 속성만 ref로 변환할 수 있습니다.
    const title = toRef(props, 'title')
  }
}
```

### 3-3.  Setup Context[​](https://ko.vuejs.org/api/composition-api-setup.html#setup-context) <a href="#setup-context" id="setup-context"></a>

`setup` 함수에 전달되는 두 번째 인자는 **Setup Context** 객체로 . 컨텍스트 객체는 `setup` 내부에서 유용할 수 있는 다른 값을 노출하다.:&#x20;

```
export default {
  setup(props, context) {
    // 속성 (비-반응형 객체, $attrs에 해당함)
    console.log(context.attrs)

    // 슬롯 (비-반응형 객체, $slots에 해당함)
    console.log(context.slots)

    // 이벤트 발송 (함수, $emit에 해당함)
    console.log(context.emit)

    // 로컬 속성 노출 (함수)
    console.log(context.expose)
  }
}
```

#### 3-1-1. 컨텍스트 객체는 반응형이 아니며, 안전하게 분해 할당될 수 있습니다:&#x20;

```
export default {
  setup(props, { attrs, slots, emit, expose }) {
    ...
  }
}
```

* `attrs`와 `slots`는 컴포넌트가 업데이트될 때, 항상 업데이트되는 스테이트풀(stateful) 객체이
  * 즉, 구조를 분해하지 말고 `attrs.x`나 `slots.x`와 같이 속성을 참조해야 한다.&#x20;
* `props`와 달리 `attrs`와 `slots`의 속성은 반응형이 아니다**.**.
  * `attrs` 또는 `slots`의 변경 사항을 기반으로 하는 사이드 이펙트는 `onBeforeUpdate` 생명 주기 훅 내에서 구현한다.

#### 3-1-2. 퍼블릭 속성 노출하기[​](https://ko.vuejs.org/api/composition-api-setup.html#exposing-public-properties) <a href="#exposing-public-properties" id="exposing-public-properties"></a>

`expose` 함수는 부모 컴포넌트가 [템플릿 참조](https://ko.vuejs.org/guide/essentials/template-refs.html#ref-on-component)를 통해 자식 컴포넌트 인스턴스에 접근할 때, 노출되는 속성을 명시적으로 제한하기 위해 사용합니다.



{% hint style="info" %}
`expose` 옵션을 사용하여 하위 인스턴스에 대한 접근을 제한할 수 있다.

```javascript
export default {
  expose: ['publicData', 'publicMethod'],
  data() {
    return {
      publicData: 'foo',
      privateData: 'bar'
    }
  },
  methods: {
    publicMethod() {
      /* ... */
    },
    privateMethod() {
      /* ... */
    }
  }
}
```

템플릿 ref를 통해 위 예제의 컴포넌트를 참조하는 부모 컴포넌트는 `publicData` 및 `publicMethod`에만 접근할 수 있다.
{% endhint %}

```
export default {
  setup(props, { expose }) {
    // 인스턴스를 "닫힘" 상태로 설정
    // 예: 부모 컴포넌트에 아무것도 노출하지 않으려는 경우
    expose()

    const publicCount = ref(0)
    const privateCount = ref(0)
    // 선택적으로 로컬 상태를 노출
    expose({ count: publicCount })
  }
}
```

### 3-4. 렌더 함수와 함께 사용하기[​](https://ko.vuejs.org/api/composition-api-setup.html#usage-with-render-functions) <a href="#usage-with-render-functions" id="usage-with-render-functions"></a>

`setup`은 범위 내 선언된 반응형 상태에 직접 접근할 수 있는 렌더 함수를 반환할 수 있다.

{% hint style="info" %}
**렌더함수란**&#x20;

Vue는 대부분의 경우에 템플릿을 사용하여 애플리케이션을 구축하는 것을 권장합니다. 그러나 JavaScript의 프로그래밍적인 기능이 필요한 경우가 있습니다. 이때 사용할 수 있는 것이 **렌더 함수**입니다.
{% endhint %}

```
import { h, ref } from 'vue'

export default {
  setup() {
    const count = ref(0)
    return () => h('div', count.value)
  }
}
```

렌더 함수를 반환하면, 다른 것을 반환할 수 없다. 내부적으로는 문제가 되지 않지만, 템플릿 참조를 통해 이 컴포넌트의 메서드를 부모 컴포넌트에 노출하려는 경우, 문제가 될 수 있습니다.

이럴 때는 `expose()`를 호출하여 이 문제를 해결할 수 있습니다:

```javascript
import { h, ref } from 'vue'

export default {
  setup(props, { expose }) {
    const count = ref(0)
    const increment = () => ++count.value

    expose({
      increment
    })

    return () => h('div', count.value)
  }
}
```

이제 `increment` 메서드는 템플릿 참조를 통해 부모 컴포넌트에서 사용할 수 있다.
