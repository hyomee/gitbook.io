# Router

Vue 라우터는 Vue.js의 공식 라우터로 다음과 같은 기능을 이용하여 싱글 페이지 앱을 쉽게 만들 수 있도록 도와 줍니다.

* 중첩된 라우트/뷰 매핑
* 모듈화된, 컴포넌트 기반의 라우터 설정
* 라우터 파라미터, 쿼리, 와일드카드
* Vue.js의 트랜지션 시스템을 이용한 트랜지션 효과
* 세밀한 네비게이션 컨트롤
* active CSS 클래스를 자동으로 추가해주는 링크
* HTML5 히스토리 모드 또는 해시 모드(IE9에서 자동으로 폴백)
* 사용자 정의 가능한 스크롤 동작

## 1. 설치

```bash
npm init vue@latest
```

vite 초기 설정시 Vue Router를 Yes로 하면 자동으로 설치 됩니다.

<figure><img src="../.gitbook/assets/image (236).png" alt=""><figcaption></figcaption></figure>

#### 1-1. 별도 설치&#x20;

```bash
npm install vue-router@4

or 

// next 레파지토리에서 받음
npm install vue-router@next
```

## 2. 라우터 구성&#x20;

라우터의 구성은 RouterRecordRaw의 배열로 URL과 컴포넌트의 관계를 정의 해야 합니다.

* path
* redirect
* alias
* name
* beforeEnter
* props
* meta
* children







참고 : [Vue Router](https://router.vuejs.org/)
