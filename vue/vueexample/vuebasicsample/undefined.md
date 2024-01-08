# 조건부 렌더링

조건부 렌더링 지시문은 true 또는 false를 값으로 반환하는 Javascript 식을 허용합니다. 그런 다음 Vue는 평가 출력에 따라 요소를 렌더링하거나 렌더링하지 않늗다.

* <mark style="color:red;">**주의 사항**</mark>&#x20;
  *   조건부 지시문이 있는 요소는 Vue가 연결된 블록으로 간주하기 위해 서로 직접 연결되어야 한다. ( 오류 발생 )\
      \


      <figure><img src="../../../.gitbook/assets/image (205).png" alt=""><figcaption><p>오류 </p></figcaption></figure>

## 1. v-if

건부로 렌더링하려는 템플릿의 요소에 연결한다.

{% code lineNumbers="true" %}
```html
<script setup>
    import { ref } from 'vue'    
    const num = ref(0);
</script>

<template>   
    숫자 입력 : <input type="text"
           v-bind:value="num"
           @input="num = $event.target.value" /> {{  num }}
    <p v-if="num === 0">숫자는 0 입니다.</p> 
</template>
```
{% endcode %}

| 초기 로딩                                                                            | 텍스트 입력                                                                           |
| -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| <img src="../../../.gitbook/assets/image (200).png" alt="" data-size="original"> | <img src="../../../.gitbook/assets/image (202).png" alt="" data-size="original"> |

* 3 line :  페이지 로딩 시 초기값으로 ref  변수로  num이  0으로 할당 되고 11 line이 표시된다.
* 화면에서 입력 박스에 숫자를 입력 하는 num 변수가 0이 아니므로 화면에서 제거 된다.
  * HTML에서 코드를 완전히 제거된다  ( CSS로 코드를 숨기는 것이 아님)

## 2. v-else-if

v-else-if 지시문은 단독으로 사용할 수 없으며 별도의 요소에서 v-if 바로 뒤에 와야 한다.

{% code lineNumbers="true" %}
```html
<script>
    export default {
        data() {
            return {
            num: 1
            }
        }
    }
</script>

<template>       
    숫자 입력 : <input type="text"
           v-bind:value="num"
           @input="num = $event.target.value" /> 
    <p v-if="num == 0">숫자는 0 입니다.</p>
    <p v-else-if="num > 0">숫자는 {{num}} , 양수</p>
    <p v-else-if="num < 0">숫자는 {{num}} , 음수</p>
</template>
```
{% endcode %}

| 양수 입력                                                                            | 음수 입력                                                                            |
| -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| <img src="../../../.gitbook/assets/image (203).png" alt="" data-size="original"> | <img src="../../../.gitbook/assets/image (204).png" alt="" data-size="original"> |

* 11 \~ 17 line : 양수, 음수, 0을 입력 하면 조건에 따라 변경되는 것을 확인 할 수 있다.

## 3. v-else&#x20;

{% code lineNumbers="true" %}
```html
<script>
    export default {
        data() {
            return {
            num: 1
            }
        }
    }
</script>

<template>       
    숫자 입력 : <input type="text"
           v-bind:value="num"
           @input="num = $event.target.value" />
    <p v-if="num == 0">숫자는 0 입니다.</p> 
    <p v-else-if="num > 0">숫자는 {{num}} , 양수</p>
    <p v-else-if="num < 0">숫자는 {{num}} , 음수</p>
    <p v-else>숫자가 아닙니다</p>
</template>
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (206).png" alt=""><figcaption></figcaption></figure>

## 4. template if 적용

v-if 만으로 화면을 구성 하면 불필요한 코드를 추가하고 DOM 깊이를 증가 하고 스타일링이 깨질 수 있으므로 template 에 if를 적용하여 간결히 할 수 있다.

* 중첩 템플릿을 사용함

<pre class="language-html"><code class="lang-html">&#x3C;script setup>
    import { ref } from 'vue'    
    const num = ref(0);
    const isNumber = ref(true);
&#x3C;/script>

&#x3C;!-- Option API 사용시
    &#x3C;script>
        export default {
            data() {
                return {
                num: 0,
                isNumber : true
                }
            }
        }
<strong>    &#x3C;/script> 
</strong>-->

&#x3C;template> 
    &#x3C;button @click="isNumber = !isNumber">전환&#x3C;/button>
    &#x3C;template v-if="isNumber">
        
        숫자 입력 : &#x3C;input type="text"
           v-bind:value="num"
           @input="num = $event.target.value" />
        &#x3C;p v-if="num == 0">숫자는 0 입니다.&#x3C;/p> 
        &#x3C;p v-else-if="num > 0">숫자는 {{num}} , 양수&#x3C;/p>
        &#x3C;p v-else-if="num &#x3C; 0">숫자는 {{num}} , 음수&#x3C;/p>
        &#x3C;p v-else>숫자가 아닙니다&#x3C;/p>
    &#x3C;/template>  
    &#x3C;template v-else>        
        문자 : &#x3C;input type="text"
           v-bind:value="num"
           @input="num = $event.target.value" /> {{num}}
    &#x3C;/template>     
&#x3C;/template>


</code></pre>

전환 버튼을 클릭하면 템플릿이 전환되는 것을 확인 할 수 있다.

| 전환버튼 isNumber = true                                                             | 전환버튼 isNumber = false                                                            |
| -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| <img src="../../../.gitbook/assets/image (207).png" alt="" data-size="original"> | <img src="../../../.gitbook/assets/image (208).png" alt="" data-size="original"> |
