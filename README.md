# JavaScript

자바스트립트는 웹 페이지의 보조 역활로 HTML 페이지에 생명력을 넣어주는 것으로 출발 하여 ECMAScript라는 표준 고유 명세를 가지게된 언어입니다.&#x20;

## 1. ECMAScript

ECMAScript는 [ECMA-262](https://ecma-international.org/publications-and-standards/standards/ecma-262/) 을 의미하며 각 브리우저 제조사는 ECMAScript사양을 준수해서 브라우저에 내장되는 자바스크립트 엔진을 구현합니다. ECMA-262에는 로그래밍 언어의 값, 타입, 객체와 프로퍼티, 함수, 표준 빌트인 객체 등 핵심 문법을 규정하고 있습니다. [자세한 스펙은 여기를 참고하세요.](https://tc39.es/ecma262/)

## 2. JavaScript

웹 브라우저에서 동작하는 유일한 프로그래밍 언어로 ECMAScript와, 브라우저가 별도로 지원하는 클라이언트 사이드 Web API, 즉 DOM, BOM, Canvas, XMLHttpRequest, fetch, requestAnimationFrame, SVG, Web Storage, Web Component, Web Worker등을 아우르는 개념입니다.

## 3.  vscode에서 html 편집

#### vscode에 Live Server를 설치 합니다.

<figure><img src=".gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

#### 1. Vscode 에서 index.html을 생성 한다

#### 2. html:5 을 에디터 창에서 입력한다.

```html
html:5
```

자동으로 다음과 같은 코드가 생성 됩니다.

```html
<!DOCTYPE html>
<html lang="kr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    
</body>
</html> 
```

#### 3. 다음과 같이 수정 합니다.

```html
<!DOCTYPE html>
<html lang="kr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HTML</title>
</head>
<body>
    <h1>자바 스트립트 학습</h1>
</body>
</html>
```

#### 4. 편집창에서 마우스 오른쪽을 누른 후 Open with Live Server 을 선택 합니다.

<figure><img src=".gitbook/assets/image (60).png" alt="" width="563"><figcaption><p>라이브 서버 활성화 하기</p></figcaption></figure>

* 라이브 서버가 활성화 됩니다. ( 5500 포트 )

<figure><img src=".gitbook/assets/image (62).png" alt=""><figcaption><p>라이브 서버 확인 </p></figcaption></figure>

#### 5. 웹 브라우저에 다음과 같은 것을 확인 할 수 있습니다.

<figure><img src=".gitbook/assets/image (61).png" alt="" width="563"><figcaption></figcaption></figure>
