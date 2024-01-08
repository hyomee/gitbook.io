# 반응형



참고 : vue 공식 문서 : [반응형 기초](https://ko.vuejs.org/guide/essentials/reactivity-fundamentals.html)&#x20;

참고 : vue 공식 문서 :[ reactive()](https://ko.vuejs.org/guide/typescript/composition-api.html#typing-reactive)

## 1. Refs

Vue3 이전에는 뷰 템플릿의 DOM 또는 컴포넌트를 가리키는 속성이었지만, Vue3에서는 reactive reference를 의미합니다.

1. `<script>`에 `import { ref } from "vue"`;을 임포트합니다.
2. 변수를 선언하고 값을 `ref()`로 감싸줍니다.
3. 변수 값을 변경할 때 `변수.value`에 변경할 값을 넣어줍니다.
4. 참고 : vue 공식문서   [플템릿참조](https://ko.vuejs.org/guide/essentials/template-refs.html#refs-inside-v-for)

<table><thead><tr><th width="134">함수</th><th>설명</th></tr></thead><tbody><tr><td>ref</td><td>전달받은 기본 자료형 변수를 반응형 객체로 변환하는 함수로 .value 속성을 사용하여 값을 변경 할 수 있습니다.</td></tr><tr><td>unref</td><td>ref로 선언한 반응형 객체를 일반 변수로 바꾸는 함수 입니다.</td></tr><tr><td>toRef</td><td>reactiveAPI로 생성한 객체의 속성을 ref 객체로 생성하는 함수 입니다.<br>toRef(object, 'property')</td></tr><tr><td>toRefs</td><td>toRef를 해당 객체의 모든 속성에 적용합니다.</td></tr><tr><td>isRef</td><td>ref객체 인지 확인 합니다.</td></tr><tr><td>customRef</td><td><ul><li><code>track</code>와 <code>trigger</code>를 인자로 수신하며, <code>get</code>과 <code>set</code> 메서드가 포함된 객체를 반환하는 사용자 지정 ref를 만들 떄 사용합니다.</li><li><code>track()</code>은 <code>get()</code> 내부에서 호출되어야 하고, <code>trigger()</code>는 <code>set()</code> 내부에서 호출되어야 하지만, 호출 조건 및 타이밍은 완전히 커스텀 제어 가능합니다.</li></ul></td></tr><tr><td>shallowRef</td><td>ref()의 얕은 버전을 참조된 객체가 통째로 변경될 때 반응형으로 작동합니다.<br>state.value = { count: 2 } : 반응형으로 됨<br>state.value.count = 2 : 반응형으로 되지 않음</td></tr><tr><td>tiggerRef</td><td>shallowRef로 참조된 객체에 대한 업데이트를 강제로 합니다.</td></tr></tbody></table>

## 2. **reactive**&#x20;

**프록시를 이용한 반응성 관리**

1. `<script>`에 `import { reactive } from "vue";`을 임포트합니다.
2. 변수를 선언하고 값을 `reactive()`로 감싸줍니다.
3. property 값을 변경할 때 `변수.property`에 변경할 값을 넣어줍니다.

<table><thead><tr><th width="175">함수</th><th>설명</th></tr></thead><tbody><tr><td>reactive</td><td>객체의 속성들을 반응형으로 만들때 사용합니다.</td></tr><tr><td>readonly</td><td>일반 객체나 프록시 객체를 읽기 전체를 읽기 전용으로 만듭니다.</td></tr><tr><td>isProxy</td><td>프록시 객체인지 확인 합니다.</td></tr><tr><td>isReactive</td><td>reactive로 생성된 프록시 객체인지 확인 합니다.</td></tr><tr><td>isReadonly</td><td>readonly로 생성된 프록시 객체인지 확인 합니다.</td></tr><tr><td>toRaw</td><td>reactive, readonly로 생성된 프록시 객체를 원래 객체로 원복 합니다.</td></tr><tr><td>makeRaw</td><td>makeRaw로 생성된 객체는 프록시 객체로 생성되지 않습니다.</td></tr><tr><td>swallowReactive</td><td>참조되는 객체의 중첩 객체를 제외한 직계 속성에만 reactive를 부여 합니다.</td></tr><tr><td>shallowReadonly</td><td>swallowReactive와 동일한 기능으로 readonly를 부여  합니다.</td></tr></tbody></table>

**2-1. 주의할 점**

reactive는 object, array 이외에는 사용할 수 없습니다.

## 3. 차이점 <a href="#ref-reactive" id="ref-reactive"></a>

1. 타입 제한\
   ref :  String, Number, Object 등 어떠한 타입에서도 사용할 수 있습니다.\
   reactive : Object, array, Map, Set과 같은 타입에서만 사용할 수 있습니다.
2. 접근 방식\
   ref :  .value property를 붙여 접근할 수 있습니다. \<templete>에서 변수를 명시할 때에는 붙일 필요가 없습니다.\
   reactive : `.`value를 붙일 필요가 없이 `<`templete`>`에서 사용하는 자바스크립트 문법처럼 사용하시면 됩니다.
