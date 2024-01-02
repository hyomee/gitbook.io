# 서비스워커

웹 브라우저 안에 있지만 웹 페이지와 분리되어 항상 실행되는 프로그램이다.

서비스 워커는 출처와 경로에 대해 등록하는 이벤트 기반 [워커](https://developer.mozilla.org/ko/docs/Web/API/Worker)로서 JavaScript 파일의 형태를 갖고 있습니다. 서비스 워커는 연관된 웹 페이지/사이트를 통제하여 탐색과 리소스 요청을 가로채 수정하고, 리소스를 굉장히 세부적으로 캐싱할 수 있다

참고 : [Service\_Worker\_API ](https://developer.mozilla.org/ko/docs/Web/API/Service\_Worker\_API)&#x20;

데모 : [Using Service Workers](https://developer.mozilla.org/en-US/docs/Web/API/Service\_Worker\_API/Using\_Service\_Workers#basic\_architecture)

## 1. 기본 아키텍처

### 1-1. navigator.serviceWorker .register() 에서 워커 코드 등록

* 성공하면 서비스 워커가 `ServiceWorkerGlobalScope`에서 실행됨
* 서비스 워커가 이벤트 처리할 준비가 됨

### 1-1. install

애플리케이션은 모든 것을 오프라인에서 사용할 수 있도록 준비 한다

* 항상 서비스 워커에게 전송되는 첫 번째 이벤트
* 설치 진행(installing) 과 설치 완료 후 대기 (installed)로 구분 된다.
* 서비스 워커가 설치 되면 waitUtil() 함수를 이용해서 캐시에 필요한 파일을 저장 한다.
  * 즉, 캐싱하는 프로세스를 시작하는 데 사용할 수 있음 (IndexedDB , Cache Storage .. )&#x20;
*   서비스 워커가 준비될 때 캐시를 저장하는 것으로 프리캐시(pre-cache)라 하며 waitUtil() 함수는 installing상태에서 pre-cache가 완료될 떄 까지 대기합니다.\


    <figure><img src="../.gitbook/assets/image (159).png" alt=""><figcaption></figcaption></figure>

### 1-2. activate

이전 버전의 서비스 워커에서 사용된 리소스를 정리 한다.

* 새로 설치된 서비스 워커는 이벤트를 수신
* 서비스 워커가 고유한 ID를 발급받아 브라우저에 성공적으로 등록되면 동작한다.
* 서비스 워커를 설치한 후에 업데이트 등의 이유로 캐시 제목이 변경되면 "install" 이벤트가 처음 부터 다시 발생한다.
  * 개발 단계에서는 개발자 도구 applocation / Service workers 에서 "Update on reload"를 활성화 하면 새로 고침을 하면 서비스워거 id를 제거하고 새로운 id를 발급 받아아  "install" 이벤트가 처음 부터 다시 발생한다.
* activate 이벤트는 활성 중 ( activating ) 과 활성 후 ( activate )로 나뉘어 진다.
  * 서비스 워커의 내용을 update하려면 캐시 제목과 프리 캐시 파일을 변경해 새로운 서비스 워커 ID로 새로운 캐시 내용이 설치되도록 해야 한다.

<figure><img src="../.gitbook/assets/image (160).png" alt=""><figcaption></figcaption></figure>

* 새 서비스 워커는 `skipWaiting()`을 호출하여 열려 있는 페이지가 닫힐 때까지 기다리지 않고 즉시 활성화되도록 요청한다.

### 1-3. register

1. register() : 활성화 후 서비스 워커는 이제 페이지를 제어하지만 성공한 후에 열린 페이지만 제어한다.
2. 서비스 워커의 새 버전을 가져올 때마다 이 주기가 다시 발생하고 새 버전이 활성화되는 동안 이전 버전의 나머지 부분이 정리한다.

### 1-4. 서비스 워커 라이프 사이클

<figure><img src="../.gitbook/assets/image (161).png" alt=""><figcaption></figcaption></figure>

## 2. 서비스 워커 이벤트

### 2-1. install

`ServiceWorkerGlobalScope` 인터페이스의 **`install`** 이벤트는 ServiceWorkerRegistration이 새 `ServiceWorkerRegistration.installing` 작업자를 획득할 때 발생하고 취소할 수 없으며 버블링되지 않는다.

```javascript
addEventListener("install", (event) => {});

oninstall = (event) => {};
```

<figure><img src="../.gitbook/assets/image (162).png" alt=""><figcaption></figcaption></figure>

참고 : [ServiceWorkerGlobalScope: install event](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/install\_event)

### 2-2. activate

`ServiceWorkerGlobalScope` 인터페이스의 **`activate`** 이벤트는 ServiceWorkerRegistration이 새 `ServiceWorkerRegistration.active` 작업자를 획득할 때 발생하고 취소할 수 없으며 버블링되지 않습니다.

```javascript
ddEventListener("activate", (event) => {});

onactivate = (event) => {};
```

<figure><img src="../.gitbook/assets/image (163).png" alt=""><figcaption></figcaption></figure>

참고 : [ServiceWorkerGlobalScope: activate event](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/activate\_event)

### 2-3. message

`ServiceWorkerGlobalScope` 인터페이스의 **`message`** 이벤트는 들어오는 메시지가 수신될 때 발생한다. 제어되는 페이지는 `ServiceWorker.postMessage()` 메서드를 사용하여 서비스 워커에 메시지를 보낼 수 있다. 서비스 워커는 선택적으로 제어된 페이지에 해당하는 `Client.postMessage()`를 통해 응답을 다시 보낼 수 있다.

이 이벤트는 취소할 수 없으며 버블링되지 않습니다.

```javascript
addEventListener("message", (event) => {});

onmessage = (event) => {};
```

<figure><img src="../.gitbook/assets/image (164).png" alt=""><figcaption></figcaption></figure>

참고 : [ServiceWorkerGlobalScope: message event](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/message\_event)

### 2-4. Functional events

#### 2-4-1. fetch

**`fetch`** 이벤트는 기본 앱 스레드가 네트워크 요청을 할 때 서비스 워커의 전역 범위에서 발생하며. 이를 통해 서비스 워커는 네트워크 요청을 가로채고 사용자 지정된 응답(예: 로컬 캐시에서)을 보낼 수 있다.

이 이벤트는 취소할 수 없으며 버블링되지 않다.

```javascript
addEventListener("fetch", (event) => {});

onfetch = (event) => {};
```

참고 : [ServiceWorkerGlobalScope: fetch event](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/fetch\_event)

#### 2-4-2. sync&#x20;

`ServiceWorkerGlobalScope` 인터페이스의 **`sync`** 이벤트는 `SyncManager`에 이벤트를 등록한 페이지(또는 작업자)가 실행 중일 때와 네트워크 연결을 사용할 수 있게 되는 즉시 발생합니다.

이 이벤트는 취소할 수 없으며 버블링되지 않습니다.

```javascript
addEventListener("sync", (event) => {});

onsync = (event) => {};
```

참고 :[ ServiceWorkerGlobalScope: sync event](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/sync\_event)

#### 2-4-3. push

push 이벤트는 서비스 워커가 푸시 메시지를 수신하면 서비스 워커의 전역 범위(`ServiceWorkerGlobalScope` 인터페이스로 표시)로 전송된다.

이 이벤트는 취소할 수 없으며 버블링되지 않습니다.

```javascript
addEventListener("push", (event) => {});

onpush = (event) => {};
```

참고 : [ServiceWorkerGlobalScope: push event](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/push\_event)



## 3. 예제 설명

### 3-1. 캐시 이름및 캐시할 파일

```javascript
const CACHE_NAME = 'abacus-pwa'; // 캐시제목 선언
const CACHE_LIST = [ // 캐시할 파일 선언
  '/pwa',
  '/pwa/index.html',
  '/pwa/manifest.json',
  '/pwa/images/iabacus_logo.png',  
  '/pwa/images/iabacus_logo_00.png'
];
```

* 1 line : 브라우저 Cache 에 저장 할 이름 \
  ![](<../.gitbook/assets/image (154).png>)
* 2 line : 캐시할 파일 목록\
  ![](<../.gitbook/assets/image (155).png>)

### 3-2. 서비스 워커

<details>

<summary>함수 정의</summary>

{% code lineNumbers="true" %}
```javascript
// 서비스 워커 install : 리소스를 cache 에 저장
const fnAddCache = async () => {
  try {
      const cache = await caches.open(CACHE_NAME);
      cache.addAll(CACHE_LIST);
      const skip = self.skipWaiting();
  } catch{
      console.log("error occured while caching...")
  }
};

// cach에서 조회 후 저장 
const fnPutCache = async (request, response) => {
  const cache = await caches.open(CACHE_NAME);
  await cache.put(request, response);
};

// 서비스 워커  activate : 리소스 reload 로 브라우저 호환성 체크 필요 
const fnEnableNavigationPreload = async () => {
  if (self.registration.navigationPreload) {
    // Enable navigation preloads!
    await self.registration.navigationPreload.enable();
  }
};

// 서비스 워커  activate : 리소스 reload 캐시 명 변경 시 update
const fnActivate = async () => {
  const cache_keys = await caches.keys();

  console.log(`cache ::  ${cache_keys}`);

  cache_keys.forEach( key => {
          if (key !== CACHE_NAME) {
              console.log("Service Worker 오래된 cache 삭제!")
              return caches.delete(key)
              
          }
      }
  );

  return Promise.all(cache_keys);
};

const fnCacheMatch = async (request) => {
  return await caches.match(request);
};


const fnPreloadResponseMatch = async (preloadResponsePromise) => {
  const preloadResponse = await preloadResponsePromise;
  if (preloadResponse) {
    console.info('using preload response', preloadResponse);
    putInCache(request, preloadResponse.clone());
    return preloadResponse;
  }
  return preloadResponse;
};

const fnServerFetch = async () => {
  const responseFromNetwork = await fetch(request.clone()); 
  fnPutCache(request, responseFromNetwork.clone());
  return responseFromNetwork;
};


// 데이터 요청에 따른 처리
const fnCacheFirst = async ({request, preloadResponsePromise, fallbackUrl}) => {

  // 캐시 우선 조회
  const responseFromCache = fnCacheMatch(request);
  if (responseFromCache) {
    return responseFromCache;
  }

  // preloaded response 응답
  const preloadResponse = await fnPreloadResponseMatch(preloadResponsePromise);
  if (preloadResponse) { 
    return preloadResponse;
  }

  // 서버 응답
  try {
    return await fnServerFetch(request);
  } catch (error) {
        
    // 서버에 없으면 대신 참조할 정보 
    if (fallbackUrl) {
      const fallbackResponse = await fnCacheMatch(fallbackUrl);
      if (fallbackResponse) {
        return fallbackResponse;
      }
    }
    
    // 최종 오류 
    return new Response('Network error happened', {
      status: 408,
      headers: { 'Content-Type': 'text/plain' },
    });
  }
}

// 데이터 요청에 따른 처리
const fnServerFirst = async ({ request }) => {

  const responseFromServer = await fnServerFetch(request.clone());
  if (responseFromServer) { 
    return responseFromServer;
  };

  // Cache 응답
  const responseFromCache = await fnCacheMatch(request);
  if (responseFromCache) {
    return responseFromCache;
  };

  // 없으면 에러 처리 
  return new Response('Network error happened', {
    status: 408,
    headers: { 'Content-Type': 'text/plain' },
  })
}
```
{% endcode %}



</details>

```javascript
// 1.서비스워커를 설치하고 캐시를 저장함
self.addEventListener('install', event => {
  console.log('서비스워커 : 설치(install)!'); 
  event.waitUntil ( fnAddCache() ) 
});

// 2. 고유번호 할당받은 서비스 워커 동작 시작
self.addEventListener('activate', event => {
  console.log('서비스워커 : 시작(activate)'); 
  event.waitUntil ( fnActivate()) ;
  // 아래 코드는 브라우저 호환성 체크 필요
  // event.waitUntil(fnEnableNavigationPreload());
});

// 3.데이터 요청시 네트워크 또는 캐시에서 찾아 반환 
self.addEventListener('fetch', event => {
  console.log("서비스 워커 : fetch!");
  // 서버 우선 
  event.respondWith( fnServerFirst( {
      request: event.request
    })
  );

  // cache 우선 
  // event.respondWith( fnCacheFirst( {    
  //     request: event.request,
  //     preloadResponsePromise: event.preloadResponse
  //     // fallbackUrl: './gallery/myLittleVader.jpg',
  //   })
  // );

});
```

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>
