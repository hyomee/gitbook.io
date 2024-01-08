# Vuex

컴포넌트에 변수를 생성하면 해당 변수의 생명주기는 해당 컴포넌트의 생명주기를 가지고 있습니다. 즉 컴포넌트가 삭제 되면 변수의 값도 삭제됩니다. 업무에 따라 값을 유지하고 하거나 애플리케이션 전체에 영향을 주는 값을 보관하여 사용하기 위해 전역 객체를 만들거나 웹 브라우저의 locaStorage 또는 indexDB를 사용 합니다.&#x20;



## 1. Vuex 나오게 된 배경

### 1-1. vue 단방향 패턴

vue는 State, View, Actions라는 단방향 패턴으로 데이터를 관리 합니다.

<figure><img src="../../.gitbook/assets/image (8).png" alt="" width="375"><figcaption></figcaption></figure>

* **state** : 컴포넌트 간 공유하는 데이터 속성 &#x20;
* **View** : 데이터를 표시하는 화면
* **Actions** : 사용자 입력에 따라 데이터를 변경하는 메서드

### 1-2. 문제점

공통의 상태를 공유하는 여러 컴포넌트 가 있는 경우 문제점은 다음과 같습니다.

* 여러 뷰는 같은 상태에 의존합니다.
  * 중첩된 컴포넌트는 prop 테이터를 전달하는데 이상이 발생 할 수 있어 주의가 필요합니다.
* 서로 다른 뷰의 작업은 동일한 상태를 반영해야 할 수 있습니다.
  * 직접 부모/자식 인스턴스를 참조하거나 이벤트를 통해 상태의 여러 복사본을 변경 및 동기화 하려는 등의 해결 방법을 사용해야 합니다.

### 1-3. 해결방안

컴포넌트에서 공유된 상태를 추출하고 이를 전역 싱글톤으로 관리하여 모든 컴포넌트는 트리에 상관없이 상태에 액세스하거나 동작을 트리거 할 수 있게 해야 합니다. 즉 상태 관리 패턴(state management pattern) 을 적용한 코드를 작성 하거나 라이브러리를 사용해야 하는데 외부 라이브러리 중애 하나가 Veux 입니다.

## 2. Vuex 란

Vuex는  상태 관리 패턴(state management pattern) + 라이브러리로  애플리케이션의 모든 컴포넌트에 대한 중앙 집중식 저장소(store)를 제공하여 어느 위치에서 있는 컴포넌트라도 쉽게 접근하고 사용할 수 있게 합니다.

| Props 통한 전달                                                                 | Vuex 를 사용한 전달                                                                |
| --------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| <img src="../../.gitbook/assets/image (9).png" alt="" data-size="original"> | <img src="../../.gitbook/assets/image (10).png" alt="" data-size="original"> |
| 다른 컴포넌트로 데이터를 보내려면 부모 컴포넌트를 계속 찾아 이벤트를 바인딩 시키고 다시 props로 내려주어야 합니다.         | s tore 이라는 일종의 중앙 저장소에서 관리 합니다.                                              |

Vue3의 ComponentAPI를 사용하면 Vuex와 같은 상태 관리 모듈을 만들어서 사용하면 Vuex는 필요 없습니다.

Vuex에 대한 자세한 사항은 [공식사이트를 참고하세요](https://v3.vuex.vuejs.org/kr/).&#x20;

참고  : [API 레퍼런스](https://v3.vuex.vuejs.org/kr/api)

**state, getters, mutations, actions** 총 4개의 속성을 가지고 있습니다.

<figure><img src="../../.gitbook/assets/image (7).png" alt="" width="526"><figcaption></figcaption></figure>

```javascript
import Vuex from 'vuex';
//vuex
export const store = new Vuex.Store({
    state: {},
    getters: {},
    mutations: {}
    actions: {}
});
```

### **2-1. namespaced**

Vuex 모듈 객체에 같은 이름의 속성이 있어도 서로 다르게  인식하는 것으로true/false 값이 있다.&#x20;

```javascript
namespaced: true
```

### **2-2. state**&#x20;

컴포넌트 간 공유하는 데이터들을 관리하는 역할을 합니다.

```javascript
statu: {
    name: null,
    phone: null,
    addresss: [
      { id: 1, post: '...', baseAddress: true },
      { id: 2, post: '...', baseAddress: false }
    ]
}
```

### **2-3. getters**&#x20;

state의 데이터를 전처리 해주는 반응형 속성으로 computed 속성 처럼 getter의 결과는 종속성에 따라 캐쉬되고, 일부 종속성이 변경된 경우에만 다시 재계산 됩니다.

```javascript
getters: {
    name: (state) => {
        return state.name
    },
    baseAddress: (state) => {
      return state.addresss.filter(address=> address.done)
    }
}
```

### **2-4. mutations**&#x20;

Vuex 저장소에서 실제로 상태를 변경하는 유일한 방법으로 event와 유사 합니다.&#x20;

* 각 mutations 에는 **타입** 문자열 **핸들러** 가 있습니다
* 핸들러 함수는 실제 상태 수정을 하는 곳이며, 첫 번째 전달인자로 state를받습니다.
* this.$states.`ommit('메서드', '값')`메서드로 호출할 수 있습니다.

```javascript
mutations: {
  newPhoneNum(state, newPhoneNum) {
    state.phone = newPhoneNum;
  }
}

// 호출 
store.commit('newPhoneNum', '01099998888')
```

### **2-5. actions**&#x20;

외부 컴포넌트에서 호출하는 함수를 가지고 있으며 데이터의 변경이 필요한 경우 여기에 작성한 함수를 호출 하여야 합니다. 내부적으로는 mutations 을 호출해야 합니다.

mutations  다른 점은

* 상태를 변이시키는 대신 액션으로 변이에 대한 커밋을 합니다.
* 작업에는 임의의 비동기 작업이 포함될 수 있습니다.

```
mutations: {
  newPhoneNum(state, newPhoneNum) {
    state.phone = newPhoneNum;
  }
},
actions: {
    newPhoneNum({ commit, state }, data) {
       commit('newPhoneNum', data)
    }
}

// 호출 
store.dispatch('newPhoneNum', '01099998888')
```

## 3. Vuex 예제

### 3-1. 직접 다운로드 / CDN <a href="#cdn" id="cdn"></a>

```html
<script src="/path/to/vue.js"></script>
<script src="/path/to/vuex.js"></script>
```

### 3-2. NPM

```bash
npm install vuex --save
```

### 3-3. 사용

```javascript
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)
```



참고 : [상태 관리 패턴](https://www.aoe.com/techradar/methods-and-patterns/state-management-pattern.html), [ Javascript 에서 상태관리패턴](https://dev.to/bnevilleoneill/state-management-pattern-in-javascript-sharing-data-across-components-2gkj)

