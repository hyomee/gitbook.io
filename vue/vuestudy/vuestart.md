---
description: Vue 기본 예제 분석
---

# 첫번째 프로그램

vue 학습을 하기전에 간단한 페이지 코드를 통해 vue에 대한 이해를 합니다.

## 1 . Vue 첫번째 페이지 만들기&#x20;

### 1-1. index.html

```html
<!doctype html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon"  href="/icon/iabacus_3220.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vue 3 학습</title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/js/main.js"></script>
  </body>
</html>
```

div tag와 main,js 외는 별다른 사항은 없다. div는 id값을 가지고 있는데 이곳에 main.js에서 무엇인지 아직은 모르지만 연결되다는 것을 예측 할 수 있다.

<figure><img src="../../.gitbook/assets/image (192).png" alt=""><figcaption></figcaption></figure>

### 2-2.  main.js

{% code title="/src/js/main.js" lineNumbers="true" %}
```javascript
import { createApp } from 'vue'
import '/src/assets/style.css'
import App from '/src/App.vue'

const app = createApp(App);
app.mount('#app')  
```
{% endcode %}

* 1 line : 루트 컴포넌트를 생성하기 위해 vue모듈에서 필요한 자원 ( createApp  함수)을 불러 온다.
* 2 line :  스타일을 불러온다.
* 3 line :  App.vue 파일을 App 컴포넌트를불러 온다.
* 4 line : createApp함수를 사용해서 애플리케이션 인스턴스(App)를 생성하고 html의 id값이 app인 tag에 마운트시킨다. ( '#app')

### 2-3. App.vue

{% code title="App.vue" lineNumbers="true" %}
```html
<template>
  <MyComponentmsg="Vue 3 학습" />                 
</template>

<script setup>
  import MyComponent from './components/MyComponent.vue';    
</script>

<style scoped>
</style>
```
{% endcode %}

#### 2-3-1. template &#x20;

* HTML기반으로 DOM 구성 영역
* MyComponent 컴포넌트를 사용하고 MyComponent컴포넌트의 파라메터로 msg를 넘긴다.

#### 2-3-2. script setup

&#x20;MyComponent.vue 컴포넌트를 사용하기 위해서 import 한다.

#### 2-3-3. style&#x20;

* style : CSS 스타일 코드 영역

### 2-4. FirstVite.vue

{% code lineNumbers="true" %}
```javascript
<script setup>
  import { ref, defineProps } from 'vue'             

  const props = defineProps({                        
    msg: String,
  })

  const propMsg = ref(props.msg)                     
  const count = ref(0)                               
</script>

<template>
  <h1>{{ propMsg }}</h1>

  <div class="card">
    <button type="button" @click="count++">count is {{ count }}</button>   
    <p>
      Edit
      <code>components/MyComponent.vue</code>  
    </p>
  </div>
</template>

<style scoped> 
</style>
```
{% endcode %}

* 2 line : ref 함수, defineProps  함수를 사용하기 위해서 import 한다.
  * ref 객체는 반응형이며, `.value` 속성에 새 값을 할당 하여`.value`에 대한 모든 읽기 작업이 추적되고, 쓰기 작업은 관련 이펙트를 트리거한다. 즉, 어떤 액션에 대한 리액션이 실시간으로 동작하도록 만들어준다.
* 4 line : defineProps  선언&#x20;
* 8 line : 반응형   ref 객체로 propMsg 변수 선언&#x20;
* 9 line : 반응형   ref 객체로 count 변수 선언 ( 초기값 할당 )
* 16 line :  반용형 변수 count에 적용이 되고 트리거 되어 화면에 반영됩니다.

## 2. 결과

count 버튼을 클릭 하면 자동 증가 된다.

<figure><img src="../../.gitbook/assets/image (193).png" alt=""><figcaption></figcaption></figure>
