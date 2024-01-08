# 인스턴스 생성



## 1. Vue 애플리케이션 인스턴스

Vue 애플리케이션 인스턴스는 전역으로 사용하는 구성요소를 등록 한다,

* src/main.js
* index.html&#x20;

### 1-1. index.html&#x20;

애플리케이션 진입점 파일로 Vue가 애플리케이션의 콘텐츠를 렌더링하는 곳이다.

{% code title="index.html" lineNumbers="true" %}
```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite + Vue</title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.js"></script>
  </body>
</html>

```
{% endcode %}

* 10 line :  id를 통해서 vue 요소가 렌더링할 위치
* 11 line : main.js를 모듈로 로딩&#x20;

### 1-2. main.js

vue 인스턴스를 설정하고 구성하는 역활을 하는 파일이다.

{% code title="src/main.js" lineNumbers="true" %}
```javascript
import { createApp } from 'vue'
import './style.css'
import App from './App.vue'

createApp(App).mount('#app')
```
{% endcode %}

* 1 line : 'vue' 패키지에서 createApp 메서드를 가져온다.
* 3 line : App Component 가지고 온다.
* 5 line :  App를  vue Application으로 생성하고dex.htm 에서 id 이름이 app인 곳에 바인딩 한다.
  * _mount 함수는 유효한 자바스크립트 querySelector ._

## 2. App.vue

Vue 컴포넌트로 Composition API를 사용하였다.

{% code title="App.vue" lineNumbers="true" %}
```html
<script setup>
  // HelloWorld 가져 오기
  import HelloWorld from './components/HelloWorld.vue'
</script>

<template>
  // HelloWorld 컴포넌트에 msg로 데이터 넘긴다.
  <HelloWorld msg="Vite + Vue 첫번째 예제" />
</template>

<style scoped>
</style>
```
{% endcode %}

### 2-1. setup() 함수로 변경

Composition API를 사용하는 다른 방법을 setup() 함수를 적용해야 하는데 이 경우는 _config 객체 생성 해해야 한다._ &#x20;

* _자바스트립트의 모듈을 생성 할 떄 사용하는 방법으로 export_ 키워드를 사용해서 내보내기를 해야 한다.
* 이때 default 키워드를 지정 구성 요소의 구성 개체와 같이 한 가지만 내보내야 한다.

{% code title="App.vue" lineNumbers="true" %}
```html
<script>
  import HelloWorld from './components/HelloWorld.vue'
  export default {
    components:  {
      HelloWorld
    },
    setup() {

    }
  }
</script>
```
{% endcode %}

* 4 line : 컴포넌트 내보내기 구성&#x20;

### 2-2. Option API로 변경

setup() 함수 만 사용하지 않는 것으로 작성 코드 및 방법은 2-1 과 동일 하다.

```html
<script>
  import HelloWorld from './components/HelloWorld.vue'
  export default {
    components:  {
      HelloWorld
    }
  }
</script>
```

<figure><img src="../../../.gitbook/assets/image (36).png" alt="" width="563"><figcaption></figcaption></figure>
