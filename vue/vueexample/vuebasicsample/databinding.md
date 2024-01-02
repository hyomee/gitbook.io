# 데이터 바인딩

데이터 바인딩은 구성 요소의 View(_템플릿_)와 Logic(_스크립트_) 간의 통신을 하는 것으로 세가지 유형의 바인딩이 있다.

* 단방향 데이터 바인딩
  * script to template : _String Interpolation_ 또는 _Attribute binding (_빌드 인디렉티브_)_
  * template to script : 이벤트 전달&#x20;
* 양방향 데이터 바인딩
  * script to template , template to script 동시에 적용&#x20;

## 1. 머스태시(Mustache) 바인딩

데이터 바인딩을 하기 위해서는 config 객체의 data 속성을 사용해야 한다.

```html
<template>
    <div>
        <h3>DataComponent.vue : 여기에 데이터 표시</h3>
        <h3>머스태시(Mustache) 바인딩 : {{ mustcheMsg }}</h3>
    </div>    
</template>
```

### 1-1.  Option API -  data 옵션

#### 1-1-1. 구문

* Option API은 data 속성를 통해서 바인딩.

```javascript
// 기본 
<script>
  export default {
    data: function() {
      return {        
        변수: 값
      }
    }
  }
</script>

//  ES6+ 단축
<script>
  export default {
    data() {       
      return {        
        변수: 값
      }
   }
  }
</script>

//  데이터를 쉼표로 구분된 키:값 쌍으로 저장하는 객체를 반환
<script>
export default {
  data() {
    return {
      fName: 'John',
      lName: 'Doe'
    }
  }
}
</script>
```

#### 1-1-1. Option API - data()

setup 함수 에서 return으로 내보내야 참조 할 수 있다.

```javascript
<script>    
    export default { 
        data() { 
            return {
                mustcheMsg: "Option API mustcheMsg 값 정의"
            }
        }
    }    
</script>
```

### 1-2. Composition API&#x20;

#### 1-2-1. Composition API - script setup

내부에 선언한 변수 및 함수등은 템플릿에 직접 바인딩&#x20;

```html
<script setup>     
    const mustcheMsg = "script script mustcheMsg 값 정의";
</script>
```

#### 1-2-2. Composition API - setup()

setup 함수 에서 return으로 내보내야 참조 할 수 있다.

```html
<script>    
    export default {
        setup() {
            const mustcheMsg = "setup() mustcheMsg 값 정의";

            return {
                mustcheMsg
            }
        }
    }    
</script>
```

## 2. 빌드 인디렉티브 - v-bind&#x20;

어트리뷰트에 바인딩하기 위해 Vue는 v-bind 지시문을 사용하여 작성한다.

* script to template으로 전달되는 단방향 바인딩
* 양방향을 만들기 위해서는 @input 디렉티브를 사용한다.
* 단축형으로 v-bind: 를 :bind="" 를 사용 할 수 있다.
  * v-bind:value="vbindMsg"
  * **:bind = "vbindMsg"**



### 2-1. 구문

```markup
<element v-bind:attribute="data"></element>

// 단축형
<element :attribute="data" />
```

{% hint style="info" %}
_Vue를 사용하면 요소의 모든 속성에 데이터를 바인딩할 수 있다. 여기에는 id , style , href 와 같은 속성과 disabled 와 같은 부울 속성이 포함된다._
{% endhint %}

```html
<template>
    <div>  
        <h3> v-bind 바인딩   </h3>            
        <input type="text" 
               v-bind:value="vbindMsg"
               @input="vbindMsg = $event.target.value" /> 
        <label>vbindMsg 값 : {{ vbindMsg }}</label> 
    </div>   
</template>
```

### 2-2. API 적용

#### 2-2-1  Composition API &#x20;

```javascript
<script setup>     
    const vbindMsg = "Composition API v-bind ";
</script>
```

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption><p>v-bind 를 사용한 단방향 바인딩</p></figcaption></figure>

#### -  양방향으로 변경

{% code lineNumbers="true" %}
```html
<template>
    <div>  
        <h3> v-bind 바인딩   </h3>            
        <input type="text" 
               v-bind:value="vbindMsg"
               @input="vbindMsg = $event.target.value" /> 
        <label>vbindMsg 값 : {{ vbindMsg }}</label> 
    </div>   
</template>
```
{% endcode %}

* 6 line : @input 디렉티브를 사용해서 이벤트 발생시 해당값($event.target.value)을 vbindMsg 변수에 할당한다.

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption><p>v-bind 를 사용한 단방향 바인딩을 @input으로 양방향으로 변경</p></figcaption></figure>

#### 2-2-2. setup()

```javascript
<script>    
    import { ref  } from 'vue'
    export default {   

        setup() {
            const mustcheMsg = "setup() mustcheMsg 값 정의";
            const vbindMsg = ref ("Composition API v-bind ");
            return {
                mustcheMsg,
                vbindMsg
            }
        }
    }    
</script> 
```

#### 2-3.  Option API &#x20;

```javascript
<script>    
    export default { 
        data() { 
            return {
                mustcheMsg: "Option API mustcheMsg 값 정의",
                vbindMsg: "Option API v-bind "
            }
        }
    }    
</script>
```

### 3-1.  속성 바인딩

input 태그의 disabled 속성을 사용하여 입력 금지를 한다. Vue는 html 태그 요소의 모든 속성에 데이터를 바인딩할 수 있다

{% code lineNumbers="true" %}
```html
<template>
    <div>  
        <h3> v-bind 바인딩   </h3>            
        <input type="text" 
               v-bind:value="vbindMsg" 
               v-bind:disabled="isDisabled"
               @input="vbindMsg = $event.target.value" /> 
        <label>vbindMsg 값 : {{ vbindMsg }}</label> 
    </div>   
</template>

// Composition API
<script setup>     
    import { ref  } from 'vue'

    const mustcheMsg = "script script mustcheMsg 값 정의"
    const vbindMsg = ref ("Composition API v-bind ")
    const isDisabled = true
</script>


// Option API
<script>    
    export default { 
        data() { 
            return {
                mustcheMsg: "Option API mustcheMsg 값 정의",
                vbindMsg: "Option API v-bind ",
                isDisabled: true
            }
        }
    }    
</script>
```
{% endcode %}

#### 3-1. 클래스 속성 적용

class 속성에 vue 바인딩을 하여 스타일을 동적으로 변경 할 수 있다

{% code lineNumbers="true" %}
```html
<template>
    <div>  
        <h3> v-bind 바인딩   </h3>            
        <input type="text" 
               v-bind:value="vbindMsg" 
               v-bind:disabled="isDisabled"
               @input="vbindMsg = $event.target.value" /> 
        <label v-bind:class="status">vbindMsg 값 : {{ vbindMsg }}</label> 
    </div>   
</template>

<style scoped>    
    .valid   { color: rgb(172, 115, 10) }
    .invalid { color: rgb(123, 20, 220) }
</style>

// Composition API
<script setup>     
    import { ref  } from 'vue'

    const mustcheMsg = "script script mustcheMsg 값 정의"
    const vbindMsg = ref ("Composition API v-bind ")
    const isDisabled = true
    const status = 'invalid'
</script>


// Option API
<script>    
    export default { 
        data() { 
            return {
                mustcheMsg: "Option API mustcheMsg 값 정의",
                vbindMsg: "Option API v-bind ",
                isDisabled: true
                status : 'invalid'
            }
        }
    }    
</script>
```
{% endcode %}

* 13 line : v-bind 을  이용해서 status 변수를 바인딩 한다.
* 12 line \~ 15 line : 스타일 정의 ( valid ,  invalid )
* 19 line, 31 line : status 변수의 값으로 스타일에 정의한 값을 할당 한다.

변수 값이 변경 되면 class 속성이 변경 되어 글자의 컬러가 변경되는 것을 확인 할 수 있다.

| status = 'valid'                                                               | status = 'invalid'                                                             |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| <img src="../../../.gitbook/assets/image (9).png" alt="" data-size="original"> | <img src="../../../.gitbook/assets/image (8).png" alt="" data-size="original"> |

#### 3-1-1. 조건식 바인딩

class 속성에 v-bind를 적용하고 삼항 연산자를 적용 하면 조건식 바인딩을 할 수 있다

{% code lineNumbers="true" %}
```html
<template>
    <!-- 조건식 바인딩 적용 --> 
    <label v-bind:class="isValid ? 'valid' : 'invalid'">
        vbindMsg 값 : {{ vbindMsg }}
    </label> 
</template>

<style scoped>    
    .valid   { color: rgb(172, 115, 10) }
    .invalid { color: rgb(123, 20, 220) }
</style>

// API
<script setup> 
    const isValid = true;
</script>
```
{% endcode %}

* 3 line : class 적용된 삼항 연산자 isValid가 true이면 style의 valid가 적용되고 false이면 style의 invalid 가 적용된다.

| isValid = true                                                                   | isValid = false                                                                  |
| -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| <img src="../../../.gitbook/assets/image (169).png" alt="" data-size="original"> | <img src="../../../.gitbook/assets/image (171).png" alt="" data-size="original"> |

#### 3-1-2 배열 클래스 바인딩

배열을 사용하여 class에 여러 클래스를 바인딩 할 수 있다.

{% code lineNumbers="true" %}
```html
<template>
    <p v-bind:class="['bold', 'italic', isValid ? 'valid' : 'invalid']">
        배열 바인딩 
    </p> 
</template>

<style scoped>    
    .valid   { color: rgb(172, 115, 10) }
    .invalid { color: rgb(123, 20, 220) }
    .bold    { font-weight: bolder }
    .italic  { font-style:  italic }
</style>

// API
<script setup> 
    const isValid = true;
</script>
```
{% endcode %}

* 2 line : class 속성을 v-bind 를 적용하고 배열을 사용하여   'bold', 'italic', isValid ? 'valid' : 'invalid' 세개의 클래스를 적용한다.

<figure><img src="../../../.gitbook/assets/image (172).png" alt="" width="563"><figcaption><p> </p></figcaption></figure>

#### 3-1-2  객체 클래스바인딩&#x20;

객체는 _키:값_ 쌍을 사용하여 선언 한다.

<pre class="language-html"><code class="lang-html">&#x3C;template>  
<strong>    &#x3C;p v-bind:class="{
</strong>                bold: true 
              }">객체 바인딘&#x3C;/p> 
&#x3C;/template>

&#x3C;style scoped>    
    .bold    { font-weight: bolder } 
&#x3C;/style>
</code></pre>
