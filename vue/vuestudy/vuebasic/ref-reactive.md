# ref와 reactive의 차이점

참고 : vue 공식 문서 : [반응형 기초](https://ko.vuejs.org/guide/essentials/reactivity-fundamentals.html)&#x20;

참고 : vue 공식 문서 :[ reactive()](https://ko.vuejs.org/guide/typescript/composition-api.html#typing-reactive)

## 1. **ref**

Vue3 이전에는 뷰 템플릿의 DOM 또는 컴포넌트를 가리키는 속성이었지만, Vue3에서는 reactive reference를 의미합니다.

1. `<script>`에 `import { ref } from "vue"`;을 임포트합니다.
2. 변수를 선언하고 값을 `ref()`로 감싸줍니다.
3. 변수 값을 변경할 때 `변수.value`에 변경할 값을 넣어줍니다.
4. 참고 : vue 공식문서   [플템릿참조](https://ko.vuejs.org/guide/essentials/template-refs.html#refs-inside-v-for)

## 2. **reactive**&#x20;

1. `<script>`에 `import { reactive } from "vue";`을 임포트합니다.
2. 변수를 선언하고 값을 `reactive()`로 감싸줍니다.
3. property 값을 변경할 때 `변수.property`에 변경할 값을 넣어줍니다.

**2-1. 주의할 점**

reactive는 object, array 이외에는 사용할 수 없습니다.

## 3. ref와 reactive의 차이점 <a href="#ref-reactive" id="ref-reactive"></a>

1. 타입 제한\
   ref :  String, Number, Object 등 어떠한 타입에서도 사용할 수 있습니다.\
   reactive : Object, array, Map, Set과 같은 타입에서만 사용할 수 있습니다.
2. 접근 방식\
   ref :  .value property를 붙여 접근할 수 있습니다. \<templete>에서 변수를 명시할 때에는 붙일 필요가 없습니다.\
   reactive : `.`value를 붙일 필요가 없이 `<`templete`>`에서 사용하는 자바스크립트 문법처럼 사용하시면 됩니다.
