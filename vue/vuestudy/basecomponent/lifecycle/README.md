# 생명주기

Vue 컴포넌트는 인스턴스를 생성할 때 초기화 , 감시 설정, 템플릿 컴파일, DOM 마운드을 생성 부터 소멸 까지의 생명주기에 따른 함수를 호출한다. 이 함수를 개발자가 의도적으로 코드를 삽입하여 실행 할 수 있다.

<figure><img src="../../../../.gitbook/assets/image (97).png" alt=""><figcaption><p>vue 3.0 생명주기</p></figcaption></figure>

다음은 ko.vuejs.org에서 제공하는 인스턴스 생명 주기에 대한 표이다,

<figure><img src="../../../../.gitbook/assets/image (105).png" alt="" width="563"><figcaption><p>인스턴스 생명 주기에 대한 표</p></figcaption></figure>

참고 : [옵션 API](https://ko.vuejs.org/api/options-lifecycle.html), [컴포지션 API](https://ko.vuejs.org/api/composition-api-lifecycle.html)

## 1. 생명주기 함수

<table><thead><tr><th width="183.33333333333331">옵션 API</th><th width="200">컴포지션 API </th><th>설명</th></tr></thead><tbody><tr><td>beforeCreate</td><td>setup</td><td>컴포넌트를 생성하기 전이므로 data, watch 같은 함수는 동작 하지 않는다.</td></tr><tr><td>create</td><td>setup</td><td>컴포넌트를 생성되면 호출되므로 data를 초기화 할 수 있다</td></tr><tr><td>beforeMount</td><td>onBeforeMount</td><td>render 함수를 호출 하기 전으로 renderTracked으로 관찰 할 수 있다</td></tr><tr><td>mounted</td><td>onMounted</td><td>DOM 엘리먼트로 마운트 된 후 호출되므로 엘리먼트 참조가 가능한다. 즉 ref와 같은 함수를 통해서 작성한 변수를 참조 할 수 있다. onRenderTracked 훅 호출 된다, ( <strong>가상 DOM -> DOM 반영</strong> )</td></tr><tr><td>beforeUpdate</td><td>onBeforeUpdate</td><td>데이터가 변경되어 DOM에 반영 되기 전 호출됨으로 엘리머크를참조하는수변 참조는 이전 값</td></tr><tr><td>updated</td><td>onUpdated</td><td>데이터가 변경되어 DOM에 변경 완료되는 시점에 호출하므로 참조된 변수를 제어 할 수 있다. 단 자식 노드가 있으면 nextTick을 사용해서 자식 업데이트가 완료될 때 까지 기다려야 한다.<br>update() {<br>   this.$nextTick(function() {<br>     // 모든 자식 노드 업데이트 됨<br>    })<br>}</td></tr><tr><td>beforeUnmount</td><td>onBeforeUnmount</td><td>컴포넌트가 해제 되기 전에 호출되므로 모든 기능 사용 할 수 있다.</td></tr><tr><td>unmounted</td><td>onUnmounted</td><td>컴포넌트가 해제되고 나서 호출되므로 모든 기능 사용 할 수 없다.</td></tr><tr><td>activated</td><td>onActivated</td><td>keepAlive태그는 컴포넌트가 다시 렌더링되는 것 막기 위해서 사용되는 것으로 v-is 디렉티브와 같이 사용되며 이것을 사용해서 컴포넌트의 상태가 보존되기 시작하면 호출된다.<br>&#x3C;KeepAlive><br>  &#x3C;componet :is:"activeComponent" /><br>&#x3C;/KeepAlive></td></tr><tr><td>deactivated</td><td>onDeactivated</td><td>KeepAlive 상태가 유지되다 효력이 상실되면 호출된다.</td></tr><tr><td>renderTracked</td><td>onRenderTracked</td><td>가상 DOM이 변경 될 때 마다 관찰용으로 호출 ( 개발 모드 전용이며, 서버 사이드 렌더링 중에 호출되지 않음 )<br>renderTracked(e) {<br>  console.log(e.log);<br>}</td></tr><tr><td>renderTriggered</td><td>onRenderTriggered</td><td>가상 DOM이 반영 될 때 마다 관찰용으로 호출 ( 개발 모드 전용이며, 서버 사이드 렌더링 중에 호출되지 않음 )</td></tr><tr><td>errorCaptured</td><td>onErrorCaptured</td><td>자식 컴포넌트에서 전파된 에러가 캡쳐되었을 때 호출</td></tr></tbody></table>

## 참고1. vue2와 vue3 생명 주기 차이

<figure><img src="../../../../.gitbook/assets/image (98).png" alt="" width="563"><figcaption><p>vue2와 vue3 생명 주기 차이</p></figcaption></figure>

참고 : [https://vuejs.org/guide/essentials/lifecycle.html](https://vuejs.org/guide/essentials/lifecycle.html)
