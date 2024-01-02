# 템플릿 문법

컴포넌트 인스턴스에 데이터를 렌더링된 DOM에 바인딩할 수 있는 HTML 기반 템플릿 문법이다. 즉 Vue 인스턴스의 데이터와 뷰의 관계를 선언적으로 정의하는 역할을 한다.

Mustache 문법과 v-bind 속성이 있다.

## 1. v-bind 속성

* 아이디, 클래스, 스타일 등의 <mark style="color:green;">**HTML 속성 값에 뷰 데이터 값을 연결**</mark>할 때 사용한다.
* 형식은 v-bind 속성으로 지정할 HTML 속성 앞에 접두사로 v-bind:를 붙여준다.
* v-bind: 문법을 :로 간소화할 수 있다.

## 2. Mustache 문법

* 뷰 인스턴스의 데이터를 <mark style="color:green;">**HTML 태그에 연결**</mark>하는 텍스트 삽입 방식이다.
* \{{ \}} 안에 자바스크립트 표현식을 쓸 수 있다.

```html
<template>
    <div>
        머스태시[Mustache"(이중 중괄호)"] ref :: {{ messageRef }}
        머스태시[Mustache"(이중 중괄호)"] :: {{ message }}
    </div>
</template>
```

### 2-1. Composition API

#### 2-1-1. script setup

{% code lineNumbers="true" %}
```javascript
<script setup>    
    import { ref } from 'vue'     
    const messageRef = ref(" 메세지 REF");
</script>

<script>
    export default {
        // data() { 아래와 동일 
        data: () => {
            return {
                message:"메세지"
            }
        } 
    }
</script>
```
{% endcode %}

* 1 line : composition 함수 선언
* 2 line : ref 사용을 위한 import
* 7 line : 컴포넌트 내부에서 사용하는 변수를 템플릿에 사용 하기 위해서 Option API 사용

#### 2-1-2. setup()

{% code lineNumbers="true" %}
```javascript
<script>  
    import { ref } from 'vue'        
    export default {        
        setup() { 
            const messageRef = ref("메세지 Ref");
            const message = "메세지";

            return {
                messageRef,
                message
            }
        }         
    }         
</script>
```
{% endcode %}

* 3\~4  line : option api 와 동일한 형태로 작성하는 것으로 setup 함수를 이용해서 작성&#x20;
* 5 line : ref를 사용해서 템플릿에 바로 적용되는 변수 선언 및 할당
* 13 line : setup 내부에서 사용할  변수 선언 및 할당
* 15 line : setup 외부에서 사용할 객체 선언
  * messageRef, message 변수를  템플릿에서 사용 가능&#x20;

### 2-2. Option API

```javascript
<script>  
    import { ref } from 'vue' 
       
    export default {        
        data() {
            messageRef : ref("메세지 Ref"),
            message : "메세지"
        }  
    }  
        
</script>
```

##
