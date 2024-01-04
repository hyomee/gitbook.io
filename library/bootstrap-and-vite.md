# Bootstrap & Vite

이 문서는 Vite를 사용하여 프로젝트에 Bootstrap의 CSS와 JavaScript를 포함하고 번들링하는 방법을작성한 문서 입니다.

[공식 문서는 여기를 참고해 주세요](https://getbootstrap.kr/docs/5.3/getting-started/vite/).

[_Bootstrap 5 CheatSheet_](https://bootstrap-cheatsheet.themeselection.com/) _에서 확인 할 수 있습니다._&#x20;

## 1. 설정하기

### 1.  Vite를 사용해서 Vue3 설치

{% code lineNumbers="true" %}
```bash
npm init vue@latest

Vue.js - The Progressive JavaScript Framework

√ Project name: ... vite-vue-bootstrap5
√ Add TypeScript? ... No / Yes
√ Add JSX Support? ... No / Yes
√ Add Vue Router for Single Page Application development? ... No / Yes
√ Add Pinia for state management? ... No / Yes
√ Add Vitest for Unit Testing? ... No / Yes
√ Add an End-to-End Testing Solution? » No
√ Add ESLint for code quality? ... No / Yes

Done. Now run:

  cd vite-vue-bootstrap5
  npm install
  npm run dev
  
> cd vite-vue-bootstrap5
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

### **3. Bootstrap을 설치**

드롭다운, 팝오버, 툴팁의 위치가 Bootstrap에 따라 달라지므로 Popper도 설치도 같이 설치 합니다. 만약 사용하지 않으면 @popperjs/core는 생략할 수 있습니다.

```sh
npm i --save bootstrap @popperjs/core
```

### 4. **추가 종속 요소를 설치**

Vite는 .scss, .sass, .less, .styl 및 .stylus 파일에 대한 기본 제공 지원을 제공하지만 Vite와 Bootstrap 외에도 Bootstrap의 CSS를 제대로 가져와 번들링하려면 또 다른 종속성(Sass)이 필요합니다

```bash
npm install -D sass
```

## 2. 코드 작성

1.  **vscode 를 실행**하고 생성한 폴더를 오픈 합니다.\


    <figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>
2.  **Bootstrap's CSS 삽입**   : assets/scss/styles.scss 파일을 생성 하고.Bootstrap's CSS 삽입을 하기 위해서 다음과 같이 작성 해 주세요.

    * Bootstrap의 모든 소스 Sass를 가져오기 위해 다음을 작성 합니다
    * 원하는 경우 스타일시트를 개별적으로 가져올 수도 있는데.자세한 내용은 [Sass 불러오기 문서를 참조](https://getbootstrap.kr/docs/5.3/customize/sass/)하세요.

    {% code title="assets/scss/styles.scss" lineNumbers="true" %}
    ```scss
    // Bootstrap's CSS 삽입
    // scss css 직접 : @import "bootstrap/dist/css/bootstrap.css";
    @import "bootstrap/scss/bootstrap";
    ```
    {% endcode %}
3.  **CSS를 로드하고 Bootstrap의 JavaScript를 불러오기** 위해서 src/js/main.js 파일에 다음과 같이 작성 해 주세요.

    * CSS를 로드하고 Bootstrap의 모든 JS를 임포트  합니다.
    * Popper는 Bootstrap을 통해 자동으로 임포트됩니다.

    {% code title="src/js/main.js" lineNumbers="true" %}
    ```js
    import { createApp } from 'vue'

    // CSS Import  
    import '../scss/styles.scss'

    // Bootstrap's JS 모두 Import 
    // 직접 js : import "bootstrap/dist/js/bootstrap.js";
    import * as bootstrap from 'bootstrap'

    import App from './App.vue'

    // 라우터 설정 하지 않아서 주석 처리
    // import router from './router'

    const app = createApp(App)

    // 라우터 설정 하지 않아서 주석 처리
    // app.use(router)

    app.mount('#app')
    ```
    {% endcode %}

    * 7 line : Bootstraps의 모든 js를 가져오지 않고 개별적으로 가지고 오기 위한 다면 다음과 같이 할 수 있습니다.  번들 크기를 줄일 수  있습니다. [자세한 정보는 여기를 참고해 주세요](https://getbootstrap.kr/docs/5.3/getting-started/javascript/).

    ```js
    import Alert from 'bootstrap/js/dist/alert';

    // 필요한 것 만 추가 
    import { Tooltip, Toast, Popover } from 'bootstrap';
    ```
4.  **src/App.vue** 를 다음과 같은 코드를 작성 합니다.\


    ```html
    <template>
      <div class="container py-4 px-3 mx-auto">
        <h1>Vue and Vite and Bootstrap !!! </h1>
        <button class="btn btn-primary">버튼n</button>
      </div>
    </template>

    <style lang="scss">
      h1 {
        color: green;

        &:hover {
          color: greenyellow;
        }
      }
    </style>
    ```

## 3. 실행

설정 작업이 완료 되었습니다.&#x20;

```bash
npm run dev
```

<figure><img src="../.gitbook/assets/image.png" alt="" width="550"><figcaption></figcaption></figure>
