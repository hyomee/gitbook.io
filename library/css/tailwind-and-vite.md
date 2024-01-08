# Tailwind & Vite

이 문서는 Vite를 사용하여 프로젝트에 Tailwind CSS CSS와 JavaScript를 포함하고 번들링하는 방법을작성한 문서 입니다.

공식문서를 참

Tailwind CSS는 모든 HTML 파일, JavaScript 구성 요소 및 기타 템플릿에서 클래스 이름을 스캔하고 해당 스타일을 생성한 다음 정적 CSS 파일에 쓰는 방식으로 작동합니다.

빠르고, 유연하며, 신뢰할 수 있으며 런타임이 없습니다._._&#x20;

## 1. 설정하기

### 1.  Vite를 사용해서 Vue3 설치

{% code lineNumbers="true" %}
```bash
npm init vue@latest

Vue.js - The Progressive JavaScript Framework

√ Project name: ... vite-vue-tailwind
√ Add TypeScript? ... No / Yes
√ Add JSX Support? ... No / Yes
√ Add Vue Router for Single Page Application development? ... No / Yes
√ Add Pinia for state management? ... No / Yes
√ Add Vitest for Unit Testing? ... No / Yes
√ Add an End-to-End Testing Solution? » No
√ Add ESLint for code quality? ... No / Yes

Done. Now run:

  cd vite-vue-tailwind
  npm install
  npm run dev
  
> cd vite-vue-tailwind
> npm install
> npm run dev   
```
{% endcode %}

* 22 Line : 정상 설치 확인 으로 실행 하고 웹 브라우저에서 확인 한다.

### 2. 폴더 삭제

Vite로 프로젝트를 생성하면 여러 폴더가 자동으로 생성되는데 다음 폴더의 파일을 모두 삭제 합니다.

* public
* components
* router : 설치시 Vue Router를 추가 해서 생성된 폴더&#x20;
* views

### **3.** Tailwind CSS **설치**

tailwindcss,  tailwindcss의 종속 파일과 tailwindcss 환경 파일을 생성 합니다.

```sh
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

### 4. tailwindcss 환경 파일 구성

파일의 모든 템플릿 파일에 대한 경로를 추가합니다

{% code title="tailwind.config.js" %}
```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```
{% endcode %}

{% code title="tailwind.css" %}
```javascript
@tailwind base;
@tailwind components;
@tailwind utilities;
```
{% endcode %}

## 2. 코드 작성

1. **vscode 를 실행**하고 생성한 폴더를 오픈 합니다.\
   ![](<../../.gitbook/assets/image (232).png>)
2. **CSS를 로드하기 위해**  tailwind**의 JavaScript를 불러오기** 위해서 src/js/main.js 파일에 다음과 같이 작성 해 주세요.

{% code title="src/js/main.js" lineNumbers="true" %}
```js
import { createApp } from 'vue'
import App from './App.vue'

import './assets/tailwind.css'
const app = createApp(App)

app.mount('#app')
```
{% endcode %}

3.  **src/App.vue** 를 다음과 같은 코드를 작성 합니다.

    ```javascript
    <script setup>
     import HelloWorld from './components/HelloWorld.vue'
    </script>

    <template> 
      <HelloWorld />
    </template>

    <style scoped>
     
    </style> 
    ```
4.  HelloWorld.vue을 다음과 같이 작성 합니다.

    ```html
    <template>
      <div class="text-3xl font-bold underline bg-blue-700">
        Vite Vue Tailwindcss
      </div>
    </template>
    ```

## 3. 실행

설정 작업이 완료 되었습니다.&#x20;

```bash
npm run dev
```

<figure><img src="../../.gitbook/assets/image (233).png" alt=""><figcaption></figcaption></figure>
