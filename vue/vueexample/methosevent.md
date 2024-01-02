# Methods & 이벤트 처리

## 1. 인라인 핸들러

Template 에 직접 참조 변수를 사용하는 것을 의미한다.

{% code lineNumbers="true" %}
```html
<script setup>
    import {ref} from 'vue'
    const cnt = ref(0);
</script>

<template>
    <div>
        <h1>이벤트 핸드링</h1>
        <h3>인라인 핸들러</h3>
        <button @click="cnt++">1 추가</button>
        // <button v-on:click="cnt++">인라인 핸들러 추가</button> 
        <p>숫자 값은: {{ cnt }}</p>
    </div>
</template> 
```
{% endcode %}

* 2 \~ 3 line : ref 를 사용하여 반응형 변수 cnt를 선언 하고 0으로 초기화 한다.
* 10 line : v-on 의 단축형을 사용하여 스크립트 cnt ++ 로 버튼을 클릭 하면 cnt의 값이 1씩 증가 한다.
  * \<button v-on:click="cnt++">인라인 핸들러 추가\</button> 와 동일함
* 12 line : 템플릿 문법을 사용해서 화면에 출력 한다.

| 초기 로딩                                                                         | 추가 버튼 클릭                                                                      |
| ----------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| <img src="../../.gitbook/assets/image (194).png" alt="" data-size="original"> | <img src="../../.gitbook/assets/image (195).png" alt="" data-size="original"> |

## 2. 메세드 핸들러

실제 업무 로직은 복잡 한데 인라인을 작성 하면 쉽지 않다. 그러므로 함수를 호출 하여 작성하는 것을 의미한다.

### 2-1 script setup

{% code lineNumbers="true" %}
```html
<script setup>
    import {ref} from 'vue' 
    const methodHandlerCnt = ref(0); 

    const fnAddCnt = function() {
        return this.methodHandlerCnt++;
    } 

</script>
<template>
    <div>
        <h3>메서드핸들러</h3>
        <button @click="fnAddCnt()">메서드핸들러 추가</button>
        // <button v-on:click="fnAddCnt()">메서드핸들러 추가</button>
        <p>숫자 값은: {{ methodHandlerCnt }}</p>
    </div>
</template>
```
{% endcode %}

* 2 \~ 5 line : script setup를 사용한 Composition API로 ref 변수 methodHandlerCnt 와 fnAddCnt 함수가 있다.
  * 변수와 함수는 템플릿에서 그냥 사용 할 수 있다.
  * <mark style="color:orange;">**주의 사항 :**</mark>&#x20;
    * <mark style="color:orange;">**템플릿에서 이벤트 핸들러로 사용시 메서드이름( @click="fnAddCnt")로 작성하면 "Unhandled error during execution of native event handler 오류가 발생한다.**</mark>
    * <mark style="color:orange;">**그러므로 메서드( @click="fnAddCnt()")로 작성을 해야 한다**</mark>.

| 초기 로딩                                                                         | 추가 버튼 클릭                                                                      |
| ----------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| <img src="../../.gitbook/assets/image (196).png" alt="" data-size="original"> | <img src="../../.gitbook/assets/image (197).png" alt="" data-size="original"> |

### 2-2 setup() & Option API

{% code lineNumbers="true" %}
```html
<script>
    import {ref} from 'vue' 
    export default {
        // Compsition API
        setup() {
            const methodHandlerCnt = ref(0);

            const fnAddCnt = function() {
                return this.methodHandlerCnt++;
            } 

            return {
                methodHandlerCnt,
                fnAddCnt
            }
        },

        // Option API
        data() {
            return {
                methodHandlerOptionCnt: 0
            }
        },
        methods: {
            fnOptionAddCnt() {
                return this.methodHandlerOptionCnt++;
            }
        }
    }    
</script>
<template>
    <div>
        <h2>이벤트 핸드링</h2>  
        <table>
            <tr>
                <td><button @click="fnAddCnt">setup() 추가</button></td>
                <td>숫자 값은: {{ methodHandlerCnt }}</td>
            </tr>
            <tr>
                <td><button @click="fnOptionAddCnt">Option API :  추가</button></td>
                <td>숫자 값은: {{ methodHandlerOptionCnt }}</td>
            </tr>
        </table>      
    </div>
</template>
```
{% endcode %}

* 5 \~ 15 line : Compsition API setup()를 사용해서 작성한 코드로 methodHandlerCnt, fnAddCnt 를 반환 하고 있다.
  * 36 \~ 37 line : v-on 단축형인 @click를 사용하였으면 script setup와 틀리게 메서드 이름(fnAddCnt)  or 메서드(fnAddCnt ())로 작성해도 된다.
* 19 \~ 29 line : Option API로 작성한 코드로 적용 방법은 Compsition API setup()를 사용한 방법과 동일 하다,

| 초기 로딩                                                                         | 추가 버튼 클릭                                                                      |
| ----------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| <img src="../../.gitbook/assets/image (199).png" alt="" data-size="original"> | <img src="../../.gitbook/assets/image (200).png" alt="" data-size="original"> |

## 2. 함수 파라메터 전달

자바스크립트로 작성을 하듯이 함수의 파라메터로 전달 하면 된다.

{% code lineNumbers="true" %}
```html
<script setup>
    import {ref} from 'vue' 
    //  ...  script setup 예제기존 코드 
    const helloMsg = ref(''); 

    const fnHello = function(msg) {
        return this.helloMsg = msg;
    } ;
    
</script>
<template>
    <div>
        <h1>이벤트 핸드링</h1> 
        <table>  
            <!-- ...  script setup 예제기존 코드  -->          
             <tr>
                <td><button @click="fnHello('홍길동')">파라메터 전달</button> </td>
                <td> {{ helloMsg }} 안녕</td>
            </tr>
        </table>    
    </div>
</template>
```
{% endcode %}

* 6 line : 매개변수 1개인 함수 작성&#x20;
* 17 line : 이벤트 발생시 데이터 전

| 초기 로딩                                                                         | 추가 버튼 클릭                                                                      |
| ----------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| <img src="../../.gitbook/assets/image (202).png" alt="" data-size="original"> | <img src="../../.gitbook/assets/image (203).png" alt="" data-size="original"> |

## 3. 여러 이벤트 처리

이벤트는 지시문에 지정된 순서대로 실행할 수 있는데 할 일을 쉼표로 구분하여 코드를 작성하면 된다.

```markup
<element @event="one($event), two($event)" />
```

{% code lineNumbers="true" %}
```html
<script setup>
    import {ref} from 'vue'
    
    const methodHandlerCnt = ref(0);
    const helloMsg = ref('');

    const fnAddCnt = function() {
        return this.methodHandlerCnt++;
    }  

    const fnHello = function(msg) {
        return this.helloMsg = msg;
    } ;
    
</script>
<template>
    <div>
        <h1>이벤트 핸드링</h1> 
        <table> 
            <tr>
                <td><button v-on:click="fnHello('홍길동'), fnAddCnt()">
                       여러 이벤트
                    </button> </td>
                <td> {{ helloMsg }} 안녕 ,  methodHandlerCnt {{ methodHandlerCnt }}</td>
            </tr>
        </table>    
    </div>
</template>
```
{% endcode %}

* 21 line : v-on:click="fnHello('홍길동'), fnAddCnt() " 처럼 실행 하고자 하는 순서 대로 , 로 작성한다.

<figure><img src="../../.gitbook/assets/image (204).png" alt=""><figcaption></figcaption></figure>

## 4. Javascript 이벤트 객체&#x20;

자바스크립트는 벤트의 동작을 조작하기 위한 함수들을 제공하고 한다. 예를 들어 클릭 이벤트에는 페이지에서 마우스를 클릭한 위치 또는 클릭 중에 또는 키를 눌렀는지 여부에 대한 정확한 x 및 y 좌표 구하기 등을 하기 위한 이벤트 (Event Object)를 사용하는데 이것을 vue에서는 사용하기 방법을 소개한다.

참고 : [Event reference](https://developer.mozilla.org/en-US/docs/Web/Events), [event 속성 또는 메서드](https://developer.mozilla.org/en-US/docs/Web/API/Event) .

### 4-1. 이벤트 수신&#x20;

```javascript
// 이벤트와 다른 파라메터가 있는 경우
funcName(event, other_params) {
  event.property
}

// 이벤트만 처리 하는 경
funcName() {
  // event is still
  // available
  event.property
}

<template> 
  <input type="text" @input="getInput">
</template>

methods: {
    getInput() {
      // get value of input on event object
      // and assign it to 'name' data property
      this.name = event.target.value
    }
  }

```

### 4-2. 이벤트 지시자

```markup
<element @event_name.modifier_name />
<element @keyup.page-up>

<form @submit.prevent="submitForm">
    <p>
        <button>Submit Form</button>
    </p>
</form>

@click.right
@keyup.enter
<element @keyup.13>
```

