# 전역 API

## 1. Application API

### 1-1.  [createApp()](https://ko.vuejs.org/api/application.html#createapp)

앱 인스턴스를 생성

* 첫 번째 인자는 루트 컴포넌트
* 선택적인 두 번째 인자는 루트 컴포넌트에 전달할 props

```javascript
// 루트 컴포넌트를 직접 서술하는 방식(inline)으로 사용하는 경우
import { createApp } from 'vue'

const app = createApp({
  /* 루트 컴포넌트 옵션 */
})

// 컴포넌트 불러오기(import)로 사용하는 경우
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)
```

* **참고** [가이드 - 앱 생성](https://ko.vuejs.org/guide/essentials/application.html)

### 1-2.  [createSSRApp()](https://ko.vuejs.org/api/application.html#createssrapp)

[SSR 하이드레이션](https://ko.vuejs.org/guide/scaling-up/ssr.html#client-hydration) 모드에서 앱 인스턴스를 생성

* 사용법은 createApp와 동일

### 1-3.  [app.mount()](https://ko.vuejs.org/api/application.html#app-mount)

컨테이너 엘리먼트에 앱 인스턴스를 마운트

```javascript
import { createApp } from 'vue'
const app = createApp(/* ... */)

app.mount('#app')
```

### 1-4. a[pp.unmount()](https://ko.vuejs.org/api/application.html#app-unmount)

### 1-5. [app.component()](https://ko.vuejs.org/api/application.html#app-component)

### 1-6. [app.directive()](https://ko.vuejs.org/api/application.html#app-directive)

### 1-7. [app.use()](https://ko.vuejs.org/api/application.html#app-use)

### 1-8. [app.mixin()](https://ko.vuejs.org/api/application.html#app-mixin)

### 1-9. [app.provide()](https://ko.vuejs.org/api/application.html#app-provide)

### 1-10. [app.runWithContext()](https://ko.vuejs.org/api/application.html#app-runwithcontext)

### 1-11. [app.version](https://ko.vuejs.org/api/application.html#app-version)

### 1-12. [app.config](https://ko.vuejs.org/api/application.html#app-config)

### 1-13. [app.config.errorHandler](https://ko.vuejs.org/api/application.html#app-config-errorhandler)

### 1-14. [app.config.warnHandler](https://ko.vuejs.org/api/application.html#app-config-warnhandler)

### 1-15. [app.config.performance](https://ko.vuejs.org/api/application.html#app-config-performance)

### 1-16. [app.config.globalProperties](https://ko.vuejs.org/api/application.html#app-config-globalproperties)
