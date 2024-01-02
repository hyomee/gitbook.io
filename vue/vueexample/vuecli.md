# 프로젝트 생성

## 1. 구문

```plaintext
npm create vite@latest
```

## 2. 실습

### 2-1. my-furst-app 프로젝트  생성 .

```powershell
PS D:\xxxx\project> npm create vite@latest
√ Project name: ... vue-first-app
√ Select a framework: » Vue
√ Select a variant: » JavaScript

Scaffolding project in D:\xxxx\project\vue-first-app...

Done. Now run:

  cd vue-first-app
  npm install
  npm run dev
```

* Project name :&#x20;
  * 표준 명명 규칙은 대시를 사용하여 이름에서 여러 단어를 구분하는 _kebab-case_
  * &#x20;_PascalCase_ , _camelCase_ 및 _snake\_case_ 작동하지만 나쁜 습관으로 간주
* 생성 프로젝트 실행&#x20;
  1. ```
     cd vue-first-app
     npm install
     npm run dev
     ```

### 2-2. 프로젝트 실행

```powershell
PS D:\xxxx\project> cd vue-first-app
PS D:\xxxx\project\vue-first-app> npm install

added 26 packages, and audited 27 packages in 32s

4 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
PS D:\xxxx\vue-first-app> npm run dev

> vue-first-app@0.0.0 dev
> vite

Port 5173 is in use, trying another one...

  VITE v5.0.10  ready in 1562 ms

  ➜  Local:   http://localhost:5174/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help

```

### 2-3. 웹 브라우저 보기

브라우저를 열고 주소창에  "http://localhost:5174/"을 입력 한다.

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

### 2-4. 폴더

| 폴더                                                                                        | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ----------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <img src="../../.gitbook/assets/image (13).png" alt="" data-size="original">              | <ul><li><p>node_modules : 프로젝트가 작동하는 데 필요한 모든 패키지를 저장하는 곳</p><ul><li><em>모든 패키지와 해당 종속성은 package.json 파일에 있</em></li></ul></li><li>public : 정적 이미지</li><li>index.html : 앱의 진입점</li><li>src : 웹 응용 프로그램을 만드는 데 사용하는 모든 스크립트, 구성 요소</li><li>gitignore : 커밋해서는 안 되는 파일과 폴더</li><li>package.json : <em>종속성</em> 및 <em>devDependencies</em>와 같은 프로젝트에 대한 모든 종류의 정보를 저장</li><li>README.md : 프로젝트에 대한 설명</li><li>vite.config,js : vite 설정 파일 ( bundler )</li></ul> |
| <p><strong>src 폴더</strong><br><img src="../../.gitbook/assets/image (14).png" alt=""></p> | <ul><li> asserts : 정적 정보 </li><li>componets : vue component 폴더</li><li>App.vue :  vue Main 파일</li><li>main,js : app 에 javasciptt</li><li>style.css : style</li></ul>                                                                                                                                                                                                                                                                                        |

## 3. 첫번째 App 만들기&#x20;

#### 1.  vscode 를 실행 하고 해당 폴더를 연다.

#### 2. src/App.vue 파일을 다음과 같이 수정 한다.

```html
<script setup>
  import HelloWorld from './components/HelloWorld.vue'
</script>

<template>
  <HelloWorld msg="Vite + Vue 첫번째 예제" />
</template>

<style scoped>
</style>

```

#### 3. src/components/Hellowold.vue 파일을 열고 다음과 같이 수정을 한다.

```html
<script setup>
  // ref 지시자 선언
  import { ref } from 'vue'

  // 부모에서 전달 받은 매게변수 선언
  defineProps({
    msg: String,
  })

</script>

<template>
  // 템플릿 문법을 사용하여 msg 표시시
  <h1>{{ msg }}</h1>
</template>

<style scoped>
</style>
```

#### 4. vscode에서 터미널 창를 열고 npm run dev 를 실행한다.

```powershell
npm run dev

> vue-first-app@0.0.0 dev
> vite


  VITE v5.0.10  ready in 1662 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

#### 5. 웹  브라우저에서 "http://localhost:5173/"를 주소창에 입력한다.

<figure><img src="../../.gitbook/assets/image (11).png" alt="" width="563"><figcaption></figcaption></figure>

