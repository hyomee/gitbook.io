# 옵션 API

* 프로퍼data : methods 및 mounted 같은 객체를 사용하여 컴포넌트의 로직를 정의
* 옵션으로 정의된 속성은 컴포넌트 인스턴스를 가리키는 함수 내부의 `this`에 노출
*   옵션 객체 속성\


    <table><thead><tr><th width="70">속성명</th><th>내용</th></tr></thead><tbody><tr><td>template</td><td>화면 요소를 정의하는 속성</td></tr><tr><td>data</td><td>인스턴스의 상태/데이터 </td></tr><tr><td>props</td><td>프로퍼티 선언, 즉 파라메터 변수 선언</td></tr><tr><td>filters</td><td>데이터를 형식화하는 함수 정의</td></tr><tr><td>methods</td><td>컴포넌트 인스턴스에서 사용할 메서드를 선언</td></tr><tr><td>computed</td><td>컴포넌트 인스턴스에 노출될 계산된 속성을 선언</td></tr><tr><td>created</td><td>뷰 인스턴스가 생성되자마자 실행할 로직을 정의할 수 있는 라이프 사이클 속성.</td></tr><tr><td>watch</td><td>데이터 변경 시 호출될 감시 콜백을 선언</td></tr><tr><td>emits</td><td>컴포넌트에서 내보낼 커스텀 이벤트를 선언</td></tr><tr><td>expose</td><td>부모 컴포넌트 인스턴스에서 템플릿 참조로 접근할 때, 노출될 공용 속성을 선언</td></tr><tr><td>componets</td><td>포넌트 인스턴스에서 사용 가능하도록 컴포넌트를 등록하는 객체</td></tr><tr><td>directives</td><td>컴포넌트 인스턴스에서 사용 가능하도록 지시자를 등록하는 객체</td></tr></tbody></table>

```javascript
<template> 
    <h1>{{propMsg}}</h1>
    <div class="card2">
    <button type="button" @click="fnIncOptionAPI">count is {{ count }}</button>
    <p>
      Edit
      <code>components/FirstViteOptionAPI.vue</code> to test HMR
    </p>
  </div>
</template>

<script>
export default {
    // 1) Component 프로퍼티 선언 
    props: {
        msg: String
    },

    // 2) Component에서 사용할 변수 선언 
    data: function() {
        return { 
            propMsg: this.msg,
            count: 0
        }
    },

    // 3) Component에서 사용할 함수 선언 
    methods: {
        fnIncOptionAPI() { 
            this.count ++;
        }
    }
}
 
</script>

<style scoped> 
</style>
```

## 1. 옵션 상태

### 1-1. data()

컴포넌트 인스턴스의 초기 반응형 상태를 반환하는 함수

* 일반 자바스크립트 객체 반환 합니다.
* &#x20;this.$data 로 접근 ( **this.$data. a는 this.a** )와 동일합니다
* 컴포넌트 인스턴스에서 프락시되지 않으므로.`his.$data._property`와 같은 방법으로 접근해야 합니다.
* <mark style="color:green;">**이상적으로 반환된 객체는 컴포넌트의 상태만 나타내는 일반 객체이어야 합니다**</mark>.

```html
<template> 
    <h1>{{propMsg}}</h1> // propMsg 값을바인딩
</template> 
```

```javascript
data() {
   return { 
      propMsg: this.msg,   // 부모에서 파라메터로 받은 값 설정
      count: 0
   }
}

methods: {
   fnIncOptionAPI() { 
      this.count ++;   // this.count 로 접근 
  }
} 
```

### 1-2. props:

모든 컴포넌트 props는 명시적으로 선언

* 선언 방법 : 문자 배열 또는 "key,value"의 객체
* 속성 :&#x20;
  * **type**:   String, Number, Boolean, Array, Object, Date, Function, Symbol, \
    \-  유효성 검사 : [Prop 유효성 검사](https://ko.vuejs.org/guide/components/props.html#prop-validation) 참고.\
    \- [불리언 캐스팅](https://ko.vuejs.org/guide/components/props.html#boolean-casting) 참고.
  * **default**: 기본값 설정&#x20;
  * **required**: prop이 필수인지 정의&#x20;
  * **validator**: prop 값을 유일한 인자로 사용하는 커스텀 유효성 검사 함수입니다. &#x20;

```javascript
//  "key,value"의 객체 선언
export default {
  props: {
    // 타입 체크
    height: Number,
    // 타입 체크 및 유효성 검사
    age: {
      type: Number,
      default: 0,
      required: true,
      validator: (value) => {
        return value >= 0
      }
    }
  }
}

// 문자 배열 선언 
export default {
  props: ['size', 'myMessage']
}
```

### 1-3. methods:

컴포넌트 인스턴스에서 사용할 메서드를 선언<mark style="color:red;">)</mark>

* 컴포넌트 인스턴스에서 직접 접근하거나, 템플릿 표현식에서 사용
* this를 통해서 접근하므로 <mark style="color:red;">화살표 함수를 사용하지 않도록 해야 합니다</mark>
* **참고**: [이벤트 핸들링](https://ko.vuejs.org/guide/essentials/event-handling.html)

```javascript
export default {
  data() {
    return { a: 1 }
  },
  methods: {
    plus() {
      this.a++
    }
  },
  created() {
    this.plus()
    console.log(this.a) // => 2
  }
}
```

### 1-4. computed:

컴포넌트 인스턴스에 노출될 계산된 속성을 선언&#x20;

* 참고 : [가이드 - 계산된 속성](https://ko.vuejs.org/guide/essentials/computed.html)

```javascript
export default {
  data() {
    return { a: 1 }
  },
  computed: {
    // 읽기전용
    aDouble() {
      return this.a * 2
    },
    // 쓰기가능
    aPlus: {
      get() {
        return this.a + 1
      },
      set(v) {
        this.a = v - 1
      }
    }
  },
  created() {
    console.log(this.aDouble) // => 2
    console.log(this.aPlus) // => 2

    this.aPlus = 3
    console.log(this.a) // => 2
    console.log(this.aDouble) // => 4
  }
}
```

### 1-5. watch :

데이터 변경 시 호출될 감시 콜백을 선언

* **참고**: [감시자](https://ko.vuejs.org/guide/essentials/watchers.html)

### 1-6. emits

컴포넌트에서 내보낼 커스텀 이벤트를 선언

* **참고:** [가이드 - 폴스루 속성](https://ko.vuejs.org/guide/components/attrs.html)

### 1-7. expose

부모 컴포넌트 인스턴스에서 템플릿 참조로 접근할 때, 노출될 공용 속성을 선언

* 참고 : [https://ko.vuejs.org/api/options-misc.html](https://ko.vuejs.org/api/options-misc.html)

## 2. 옵션: 렌더링

참고 : [https://ko.vuejs.org/api/options-rendering.html](https://ko.vuejs.org/api/options-rendering.html)

## 3. 옵션 : 컴포지션

### 3-1. provide

하위 컴포넌트에 주입(Inject)할 수 있도록 값을 제공(Provide)

* **참고**: [Provide / Inject](https://ko.vuejs.org/guide/components/provide-inject.html)

### 3-2.  inject

상위 컴포넌트 또는 app.provide()를 통해 앱에서 제공(Provide)된 값 중, 주입(Inject)할 속성을 선언

* **참고**: [Provide / Inject](https://ko.vuejs.org/guide/components/provide-inject.html)

## 4. 컴포넌트 인스턴스

* 참고 : [https://ko.vuejs.org/api/component-instance.html](https://ko.vuejs.org/api/component-instance.html)

## 5. 옵션: 기타

* 참고 : [https://ko.vuejs.org/api/options-misc.html](https://ko.vuejs.org/api/options-misc.html)

##

