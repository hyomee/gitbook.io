# Vue

## 1. Vue 설치

* node.js 설치
* Vue 설치 : npm install vue

## 2. Vite

번들링 통해서 개발하였을 때  개발 서버를 가동하는데 모듈이 많으면 시간이 많이 걸린다. HMR([Hot Module Replacement](https://webpack.kr/concepts/hot-module-replacement/))을 사용하여도 시간이 걸릴 수 있다. 늦은 시간을 해결하고자 하는 목적으로 만든 것이 Vite이다. **종속성** 과 **소스 코드라는** 두 가지 범주로 속도 개선을 한다.

* 종속성을 사전 번들 :  [esbuild를](https://esbuild.github.io/) 사용하여 종속성 모듈을 사전 번들하여 속도 개선
* 모든 소스 코드를 동시에 로드하지 않고 [Vite는 기본 ESM을](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) 통해 소스 코드를 제공하여 브라우저가 요청하는 대로 필요에 따라 소스코드를 변환하여 제공한다.

<figure><img src="../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

번들러  기반 빌드는 모듈이 많아 질 수록 늦어진다.

<figure><img src="../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

Vite가 기본적으로 다양한 [성능 최적화를 구현하는 사전 구성된 ](https://vitejs.dev/guide/features#build-optimizations)[빌드 명령을](https://vitejs.dev/guide/build) 제공한다.

1. node.js 설치
2. npm create vite@latest

<pre><code><strong>> npm create vite@latest
</strong>Need to install the following packages:
  create-vite@4.4.1
Ok to proceed? (y)
√ Project name: ... first-vite
√ Select a framework: » Vue
√ Select a variant: » JavaScript

Scaffolding project in C:\PRJ\VUE\vite\first-vite...

Done. Now run:

  cd first-vite
  npm install
  npm run dev
</code></pre>

3. npm run dev

```
> npm run dev

> first-vite@0.0.0 dev
> vite


  VITE v4.4.9  ready in 689 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h to show help

```

참고 : [vitejs.dev](https://vitejs.dev/guide/why.html)

## 3. Vue CLI  이용한  프로젝트  생성

1. Vue CLI 설치 : npm install -g @vue/cli
2. Vue 프로젝트 생성 :  vue create 프로젝트명
3. 실행 : npm run serve

## 4. 개발 Tool 설치

1. 다운 로드 : [https://code.visualstudio.com/download](https://code.visualstudio.com/download)
2. Visual Studio Code 실행&#x20;
3. Extension 설치: **Vetur**, **Vue Volar extension Pack**
4. devtools 설치&#x20;
   * 크롬 : [https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd)
   * Edge : [https://microsoftedge.microsoft.com/addons/detail/vuejs-devtools/olofadcdnkkjdfgjcmjaadnlehnnihnl](https://microsoftedge.microsoft.com/addons/detail/vuejs-devtools/olofadcdnkkjdfgjcmjaadnlehnnihnl)

## 5. 라이브러리 추가

* **electron :** vue add electron-builder
  * 실행 : yarn run electron:serve
* &#x20;[Vuex](https://vuex.vuejs.org/) : 상태 관리 시스템 .
* &#x20;[**Vue Router**:](https://router.vuejs.org/) 페이지 사이를 탐색하고 구성 요소를 로드.
* &#x20;[**Vue Test Utilities**](https://test-utils.vuejs.org/): 는 구성 요소와 애플리케이션을 테스트&#x20;
* &#x20;[**I18n**](https://kazupon.github.io/vue-i18n/)은 구성 요소 및 응용 프로그램에 국제화 .
* [**Jest**](https://jestjs.io/) **,** [**Mocha & Chai**](https://mochajs.org/) : 단위 테스트를 수행.
* [**Firebase**](https://firebase.google.com/) : Firestore 데이터베이스와 같은 서비스와의 통합을 보고 관리&#x20;
* [**Vuetify**](https://vuetifyjs.com/en/) **:** 는 미리 만들어진 구성 요소가 있는 Vue UI 라이브러리입니다.
* [**Tailwind**](https://tailwindcss.com/)는 가볍고 아름다운 사용자 인터페이스를 개발하는 데 도움이 되는 CSS 프레임워크&#x20;
