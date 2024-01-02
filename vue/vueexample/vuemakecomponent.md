# 컴포넌트 만들기

Vue 는 HTML, CSS 및 Javascript(또는 TypeScript)를 하나의 독립적인 파일고 결합하여 사용 하는데 이때 독립적인 파일을 컴포넌트라고 한다.

## 1.  단일 컴포넌트&#x20;

Single File Components (SFCs - .vue files)로 확장자는 .vue 파일로 다음과 같은 블럭으로 구성 된다.

* \<template>\</template> 블록 :  UI의 HTML 마크업으로 구조를 정의 하는.곳
* \<script>\</script> 블록 :  템플릿에 대한 데이터와 논리를 작성 하는 곳
* \<style>\</style> 블록 :  템플릿 블록의 마크업 스타일을    정의 하는 곳

```html
<template>
  <!-- HTML 코드 -->
</template>

<script>
  // 자바 스크립트 또는 타입 스크립트로 작성된 데이터 및 업무 코
</script>

<style>
  /* 스타일*/
</style>
```

## 2. 컴포넌트 생성

### 2-1. 템플릿 생성

#### 2-1-1. DataComponent.vue 작성

파일 이름은 PascalCase, camelCase, kebab-case 또는 snake\_case 사용할 수 있지만 PascalCase를 사용하는 것이 일반적이다.

{% code title="DataComponent.vue" lineNumbers="true" %}
```html
<template>
    <div>
        <h3>DataComponent.vue : 여기에 데이터 표시</h3>
    </div>    
</template>

<script>
</script>
```
{% endcode %}

#### 2-1-2. 컴포넌트 가져오기

컴포넌트를 가져오기 위해서는&#x20;

1. 연결된 다른 구성 요소로 구성 요소를 가져온다.
2. 구성 개체에 구성 요소를 등록한다.
3. 사용자 지정 태그를 추가하여 템플릿의 구성 요소를 사용한다.

#### 2-1-2-1 구문

```markup
<script>
import ComponentName from './path-to/ComponentName'

export default {
  components: {
    ComponentName: 'ComponentName'
  }
}
</script>

// 객체의 키와 값이 같을 때 ES6 속기 구문
<script>
import ComponentName from './path-to/ComponentName'

export default {
  components: {
    ComponentName
  }
}
</script>
```

#### **2-1-2-2.  Composition API : script setup**&#x20;

* Composition API script setup를 사용하는 경우 템플릿에서 직접 사용 가능 하기 때문에구성 개체에 구성 요소를 등록 할 필요가 없습니다. &#x20;
* 컴포넌트 경로는 다음과 같은 탐색 경로를 사용 합니다.
  * './' :  같은 폴더
  * '../' ​:  하나 상위 폴더
  * '../../' : 둘 이상 상위 폴더&#x20;

{% code title="App.vue" lineNumbers="true" %}
```html
<script setup>
  import HelloWorld from './components/HelloWorld.vue'
  // 1. 구성 요소 가져 오기
  import DataComponent from './components/DataComponent.vue'
</script>

<template>
  <HelloWorld msg="Vite + Vue 첫번째 예제" />
  // 3.  사용자 지정 태그를 추가하여 템플릿의 구성 요소를 사용
  <data-Component />
</template>

<style scoped>
</style>
```
{% endcode %}

* 9 line : 템플릿의 구성 요소를 사용할 떄는 PascalCase 또는 kebab-case를 사용할 수 있다
  * PascalCase 태그는 사용자 지정 태그 이름에 대한 W3C 규칙을 준수하지 않으므로 kebab-case만 사용하는 것이 좋다.
  * kebab-case 가 아닌 경우 vue 가 자동 변환 한다.

#### **2-1-2-3.**  Composition API : setup()   or Option API

* &#x20;config 개체에 있는 components 옵션을이용 해서 구성 개체에 구성 요소를 등록합니다.

{% code title="App.vue" lineNumbers="true" %}
```html
<script>
  import HelloWorld from './components/HelloWorld.vue'  
  import DataComponent from './components/DataComponent.vue'
  export default {
    components:  {
      HelloWorld,
      DataComponent 
    }
  }
</script>

<template>
  <HelloWorld msg="Vite + Vue 첫번째 예제" />
  <data-Component />
</template>

<style scoped>
</style> 
```
{% endcode %}





### &#x20;



