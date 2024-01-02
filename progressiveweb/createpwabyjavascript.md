# PWA 만들기

HTML과 자바스크립트로 페이지를 만들면서 PWA의 기본 개념을 학습한다.

## 1. PWA 만들기 위한 사전 작업

1. 이미지 준비
   * 홈  화면 아이콘
   * 웹 브라우저의   (favcion) 파일 &#x20;
   * 스프래시 스크린(splash screen) 이미지 파일
   * 프로그램 자체에서 사용할 이미지 파일

## 2. PWA 만들기

### 2-1. manifest.json 파일 작성

<pre class="language-json"><code class="lang-json"><strong>{
</strong>  "name": "abacus PWA",
  "short_name": "abacus",
  "description": "abacus study pwa program",
  "scope": ".",
  "start_url": "./",
  "display": "fullscreen",
  "orientation": "portrait",
  "theme_color": "#000000",
  "background_color": "#000000",
  "icons": [
    {
      "src": "images/images/iabacus_14490.png",
      "sizes": "144x90",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "images/images/iabacus_144144.png",
      "sizes": "144x144",
      "type": "image/png",
      "purpose": "any"
    }
  ]
}
</code></pre>

#### 2-1-1. manifest.json 각 요소의 의미

* name : 바로가기 아이콘 , 설치를 권장하는 팝업 배너와 스프래시 스크린에 표시되는 제목\
  ![](<../.gitbook/assets/image (152).png>)
* short\_name : 바탕화면 바로가기 아이콘 아래 표시되는 제목\
  ![](<../.gitbook/assets/image (153).png>)
* description : 애플리케이션의 소개로 웹 크롤러가 가져가는 정보
* scope : 매니페스트에 정의된 내용이 적용될 수 있는 파일의 범위&#x20;
  * " . " : 현재 위치
  * "./ " : 현재 위치를 중심으로 시작하는 하위 폴더
* start\_url : 프로그램을 시작될 URL을 루트 ( "./")로 설정&#x20;
  * 반드시 " . "으로 시작 해야 함.
* display : PWA를 실행하면 나타나는 화면을 설정하는 속성
  * fullscreen : 기기의 최대 화면 ( 지원하지 않으면 standalone )
  * standalone  : 웹 브라우저의 주소, 상태 표시줄 등 모두 제거&#x20;
  * minimal-ui : 상단 주소 표시줄만 추가 ( 지원하지 않으면 standalone )&#x20;
  * browser : 웹 브라우저와 똑같은 모
* orientataion : 화면 방향
  * portrait : 세로
  * landscape : 가로
* theme\_color : 상태 표시줄 색상
* background\_color : 스플래시 색상
* icons : 스플래시 스크린에 사용할 아이콘&#x20;
  * 128 dpu에 가장 가까운 이미지 찾아 표시
  * 144x144 하나 존재 해야 함.

### 2-2. html 작성

{% code title="index.html" lineNumbers="true" %}
```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <!-- PWA 매니페스트 파일 연결 -->
    <link rel="manifest" href="manifest.json">
    <meta name="theme-color" content="#ffffff">
    <!-- 모바일 기기 설정-->
    <meta name="viewport" content="width=device-width, initial-scale=1.0"> 
    <!-- 주소창에 파비콘 설정 -->
    <link rel="shortcut icon" href="images/icons/iabacus_3220.ico" >
</head>
<style>
    html,
    body {
      /* html, body 모두 높이를 100%로 고정시켜야 flexbox 동작 */
      height: 100%;
      background-color: #000000;
      color: #ffffff;
    }

    .container {
      height: 100%;            /* 높이를 100%로 고정시킴 */
      display: flex;           /* 컨테이너를 flexbox 스타일로 변경 */
      align-items: center;     /* 상하 가운데 정렬 */
      justify-content: center; /* 좌우 가운데 정렬*/
    }
  </style>
<body>
    <div class="container">
        <h1>PWA</h1>
        <P><img src="images/iabacus_logo_00.png"></P>
    </div>
</body>
<script type="module">
    import fnRegisterPwa from './script/mjs/pwa.mjs';
    fnRegisterPwa();
</script>
</html>
```
{% endcode %}

* 6 line : PWA 매니페스트 파일 연결
* 11 line : 주소창에 파비콘 설정
*   35 line : 서비스 워커 등록 모듈&#x20;

    {% code title="./script/mjs/pwa.mjs" lineNumbers="true" %}
    ```
    const fnLoadPwd = () => {
        // 서비스 워커 등록을 하기 위한 js 연경
        navigator.serviceWorker
                .register('./script/js/service_worker.js')
                .then((msg)=> {
                    console.log('서비스 워커 등록 :: ' + msg);
                })
                .catch((errMsg)=> {
                    console.log('서비스 워커 미 등록 ::' + errMsg);
                });
    };

    const fnRegisterPwa = ()=> {
        // 페이지 로딩 시 서비스 워커 등록
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', fnLoadPwd());        
        } else {
            console.log('PWA를 지원하지 않는 브라우저 입니다.');
        }
    };

    export default fnRegisterPwa;
    ```
    {% endcode %}

    * fnRegisterPwa :  페이지 로딩 시 서비스 워커 등록&#x20;
    * fnLoadPwd : 서비스 워커 register() 함수에  있는 install, activate, fetch 기능 등록

### 2-3. service\_worker 파일

{% code title="service_worker.js" lineNumbers="true" %}
```javascript
const CACHE_NAME = 'abacus-pwa'; // 캐시제목 선언
const CACHE_LIST = [ // 캐시할 파일 선언
  '/pwa',
  '/pwa/index.html',
  '/pwa/manifest.json',
  '/pwa/images/iabacus_logo.png',  
  '/pwa/images/iabacus_logo_00.png'
];

// 1.서비스워커를 설치하고 캐시를 저장함
self.addEventListener('install', event => {
  console.log('서비스워커 설치함!'); 
  event.waitUntil (
    (async() => {
        try {
            const cache = await caches.open(CACHE_NAME);
            cache.addAll(CACHE_LIST);

            const skip = self.skipWaiting();
        }
        catch{
            console.log("error occured while caching...")
        }
    })()
  );
  console.log('서비스워커 설치함!!!!');
});

// 2. 고유번호 할당받은 서비스 워커 동작 시작
self.addEventListener('activate', event => {
  console.log('서비스워커 동작 시작됨!');
  
  (async () => {

    const cache_keys = await caches.keys()
    console.log(cache_keys)

    cache_keys.forEach(
        key => {
            if (key !== CACHE_NAME) {
                console.log("Service Worker 오래된 cache 삭제!")
                return caches.delete(key)
                
            }
        }
    )
    return Promise.all(cache_keys)


})()
});

// 3.데이터 요청시 네트워크 또는 캐시에서 찾아 반환 
self.addEventListener('fetch', event => {
  console.log("Service Worker : fetch!")
    event.respondWith(
        // 서버 응답이 없으면 casche 정보 조회
        fetch(event.request)
        .catch(() => {
            caches.match(event.request)
        })
        
    )
});
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption><p>화면</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption><p>서비스워커</p></figcaption></figure>

**소스** :  [https://github.com/hyomee/Javascript/tree/main/pwa](https://github.com/hyomee/Javascript/tree/main/pwa)

