# 컴포넌트

Vue에서 컴포넌트는 재 사용가능한 블럭을 의미 하는것으로 미니 앱 입니다.

* 기본 코드 bootstrap을 사용해서 작성한 코드 입니다.

<details>

<summary><strong>기본코드 :  bootstrap을 사용해서 작성한 코드 입니다.</strong></summary>

```
<ul class="container list-group list-unstyled w-50 "   >
  <li class="border-3  border-bottom">
    <h2 class="p-1 text-center fw-bold">홍길동</h2> 
    <button type="button"
            class="btn btn-primary btn-sm">상세정보 보기</button>
    <ul class="list-unstyled">
      <li><strong>전화번호:</strong> 010 9321 1234</li>
      <li><strong>이메일:</strong> hong01@gmail.com</li>
      <li><strong>주소:</strong> 서울 송파구 잠실동</li>
    </ul>
  </li>
  <li class="border-3 border-bottom">
    <h2 class="p-1 text-center fw-bold">김바둑</h2>
    <button type="button"
            class="btn btn-primary btn-sm">상세정보 보기</button>
    <ul class="list-unstyled">
      <li><strong>전화번호:</strong> 010 7239 2345</li>
      <li><strong>이메일:</strong> kim01@gmail.com</li>
      <li><strong>주소:</strong> 서울 송파구 삼전동</li>
    </ul>
  </li>
</ul>
```

* 2 \~ 11 line : 12 \~ 23 line와 같은 기능을 하는 것으로 v-for를 사용하여 컴포넌트를 만들 수 있습니다.

</details>

<figure><img src="../../../.gitbook/assets/image (239).png" alt=""><figcaption><p>기본 코드 결과</p></figcaption></figure>

## 1. 컴포넌트 만들기

기본 코드를반응형 변수와 v-for문을 사용하는 코드로 변경을 합니다.

<details>

<summary>v-for로 변경</summary>

```html
<script setup>
  import { ref } from 'vue'

  const customers = ref([
    { 
      id : 1, 
      name: '홍길동',
      phone: '010 9321 1234',
      email: 'hong01@gmail.com',
      address: '서울 송파구 잠실동'
    },
    { 
      id : 2, 
      name: '김바둑',
      phone: '010 7239 2345',
      email: 'kim01@gmail.com',
      address: '서울 송파구 삼전동'
    }
  ])
</script>
<template>   
  <div class="container  w-50 p-2" >
    <h1 class="m-2 p-2  mb-1 bg-secondary text-white">v-for 변경</h1>
    <ul class="container list-group list-unstyled  ">
      <li class="border-3  border-bottom"
          v-for="customer in customers" :key="customer.id">
        <h2 class="p-1 text-center fw-bold">{{customer.name}}</h2> 
        <button type="button"
                class="btn btn-primary btn-sm">상세정보 보기</button>
        <ul class="list-unstyled">
          <li><strong>전화번호:</strong>{{customer.phone}}</li>
          <li><strong>이메일:</strong>{{customer.email}}</li>
          <li><strong>주소:</strong>{{customer.address}}</li>
        </ul>
      </li>
    </ul>
  </div>
</template>
```

</details>

<details>

<summary>상세보기 버튼 기능 추가 ( Component_01.vue )</summary>

{% code title="Component_01.vue" lineNumbers="true" %}
```html
<script setup>
  import { ref } from 'vue';

  const customers = ref([
    { 
      id : 1, 
      name: '홍길동',
      phone: '010 9321 1234',
      email: 'hong01@gmail.com',
      address: '서울 송파구 잠실동'
    },
    { 
      id : 2, 
      name: '김바둑',
      phone: '010 7239 2345',
      email: 'kim01@gmail.com',
      address: '서울 송파구 삼전동'
    }
  ]);

  const isDetailVisible = ref(false);

  const fnToggleDetail = function() {
    this.isDetailVisible = !this.isDetailVisible; 
  }
</script>
<template>   
  <div class="container  w-50 p-2" >
    <h1 class="m-2 p-2  mb-1 bg-secondary text-white">v-for 변경</h1>
    <ul class="container list-group list-unstyled  ">
      <li class="border-3  border-bottom"
          v-for="customer in customers" :key="customer.id">
        <h2 class="p-1 text-center fw-bold">{{customer.name}}</h2> 
        <button type="button"
                class="btn btn-primary btn-sm"
                @click="fnToggleDetail()">
                상세정보 {{isDetailVisible ? '숨기기' : '보기'}}
        </button>
        <ul class="list-unstyled" v-if="isDetailVisible">
          <li><strong>전화번호:</strong>{{customer.phone}}</li>
          <li><strong>이메일:</strong>{{customer.email}}</li>
          <li><strong>주소:</strong>{{customer.address}}</li>
        </ul>
      </li>
    </ul>
  </div>
</template>
```
{% endcode %}

</details>

### 1-1. 재사용 컴포넌트 만들기

#### 1-1-1. 상세보기 버튼의 문제점

상세 보기 버튼을 클릭하면 모두 같이 변경됩니다.

* FADSF
* FSDF



| 초기 로딩                                                                          | 상세보기 클릭                                                                        |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| <img src="../../../.gitbook/assets/image (2).png" alt="" data-size="original"> | <img src="../../../.gitbook/assets/image (1).png" alt="" data-size="original"> |

Component\_01.vue에서 상세 버튼을 클릭하면 해당 정보만 보여주기 위해서는 fnToggleDetail() 함수에 파라메터를 넘겨서 로직을 수정 하고 template 영역도 수정 해야 합니다.&#x20;

만약  고객(customer)에 2개 이상의 객체가 있다면 Boolean 변수 (isDetailVisibleRefA , isDetailVisibleRefB ) 이 많이 필요하게 됩니다.&#x20;

```javascript
const isDetailVisibleRefA = ref(props.isDetailVisible);
const isDetailVisibleRefB = ref(props.isDetailVisible);

const fnToggleDetail = function(id) {  
    if (id === "1") {
        this.isDetailVisibleRefA = !this.isDetailVisibleRefA; 
    }

    if (id === "2") {
        this.isDetailVisibleRefB = !this.isDetailVisibleRefB; 
    }   
}
```

이것은 로직이 복잡해 지고 확장성에 문제가 있습니다. 만약 중복 되는 부분을 컴포넌트로 만들어 재 사용하면 이 문제를 해결할 수 있습니다.

#### 1-1-2. 재 사용 컴포너트 생성

Component\_01.vue 예제를 다음과 같이 수정 합니다.&#x20;

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>ComponentView.vue 소스</summary>

<pre class="language-html" data-line-numbers><code class="lang-html"><strong>&#x3C;script setup>
</strong>  import { ref } from 'vue';
  import Component_00 from '@/components/rtComponent/Component_00.vue';
  import Component_01 from '@/components/rtComponent/Component_01.vue';
  import ComponentReUse from '@/components/rtComponent/ComponentReUse.vue';

  const customers = ref([
    { 
      id : 1, 
      name: '홍길동',
      phone: '010 9321 1234',
      email: 'hong01@gmail.com',
      address: '서울 송파구 잠실동'
    },
    { 
      id : 2, 
      name: '김바둑',
      phone: '010 7239 2345',
      email: 'kim01@gmail.com',
      address: '서울 송파구 삼전동'
    }
  ]);

  const isDetailVisible = ref(true)

&#x3C;/script>

&#x3C;template>
    &#x3C;component_00 />
    &#x3C;component_01 />
    &#x3C;div class="container  w-50 p-2" >
        &#x3C;h1 class="m-2 p-2  mb-1 bg-secondary text-white">Component ReUse&#x3C;/h1>
        &#x3C;ul class="container list-group list-unstyled ">
            &#x3C;component-re-use />  
        &#x3C;/ul>
    &#x3C;/div>
&#x3C;/template>
</code></pre>

* 31 \~ 41 Line : div로 영역을 만들고 ul 영역을 v-for를 이용해서 ComponentReUse 컴포넌트를 삽입합니다.



</details>

{% code title="ComponentReUse.vue" lineNumbers="true" %}
```javascript
<script setup>
    import {  ref } from 'vue';
    const isDetailVisibleRef = ref(true); 

    const fnToggleDetail = function(id) {  
        this.isDetailVisibleRef = !this.isDetailVisibleRef;  
    }     
</script>
<template>    
    <li class="border-3  border-bottom" >
        <h2 class="p-1 text-center fw-bold">{{customer.name}}</h2> 
        <button type="button"
                class="btn btn-primary btn-sm"
                @click="fnToggleDetail(customer.id)">
                상세정보 {{isDetailVisibleRef ? '숨기기' : '보기'}}
        </button>
        <ul class="list-unstyled" v-if="isDetailVisibleRef">
            <li><strong>전화번호:</strong>{{customer.phone}}</li>
            <li><strong>이메일:</strong>{{customer.email}}</li>
            <li><strong>주소:</strong>{{customer.address}}</li>
        </ul> 
    </li>  
</template>
```
{% endcode %}

* 11  line : customer를 참조 할 수 없다는 오류가 발생 합니다.

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

해당 오류를 수정하기 위해서 컴포넌트에서 하위 컴포넌트에 데이터를 넘기기 위한 props를 사용해여 합니다.

## 2. 부모에서 자식으로 ( props )

부모 컴포넌트에서 자식 컴포넌트로 겍체를 넘기기 위해서는 props를 사용해야 합니다. [자세한 것은 공식 사이트를 참고해 주세요.](https://vuejs.org/guide/components/props.html)

<details>

<summary>props에서 지원되는 타입</summary>

* **Number**
* **Boolean**
* **Array**
* **Object**
* **Data**
* **Function**
* **Symbol**

</details>

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

{% code title="ComponentView.vue" lineNumbers="true" %}
```html
<script setup>
  import { ref } from 'vue';
  import Component_00 from '@/components/rtComponent/Component_00.vue';
  import Component_01 from '@/components/rtComponent/Component_01.vue';
  import ComponentReUse from '@/components/rtComponent/ComponentReUse.vue';

  const customers = ref([
    { 
      id : 1, 
      name: '홍길동',
      phone: '010 9321 1234',
      email: 'hong01@gmail.com',
      address: '서울 송파구 잠실동'
    },
    { 
      id : 2, 
      name: '김바둑',
      phone: '010 7239 2345',
      email: 'kim01@gmail.com',
      address: '서울 송파구 삼전동'
    }
  ]);

  const isDetailVisible = ref(true)

</script>

<template>
    <component_00 />
    <component_01 />
    <div class="container  w-50 p-2" >
        <h1 class="m-2 p-2  mb-1 bg-secondary text-white">Component ReUse</h1>
        <ul class="container list-group list-unstyled ">
            <component-re-use                                
                isDetailVisible
                v-for="customer in customers"
                :key = "customer.id"
                :customer= customer
                 ></component-re-use> 
        </ul>
    </div>
</template>
```
{% endcode %}

* 35 line : 상세 정보를 toggle 하기 위한 isDetailVisible boolean  값 전달
* 36 line : customers 객체 정보를 v-for문을 통해사 component-re-use 컴포넌트의 파라메터로 값 전달달

{% code title="ComponentReUse.vue" lineNumbers="true" %}
```javascript
<script setup>
    import {  ref } from 'vue';

    // props 불변 이다,. vue는 단방향이다.
    const props = defineProps({                    
        isDetailVisible: {
            type: Boolean,   
            required: true, 
            default: false
        },      
        customer: {
            id: {
                type: Number,
                required: true
            } 
        }
    }) 

    // props는 단방향으로 변경을 하기 위해서는 반응형으로 변경해야 한다.
    const isDetailVisibleRef = ref(props.isDetailVisible);

    // 상세보기 클릭시 
    const fnToggleDetail = function() {  
        this.isDetailVisibleRef = !this.isDetailVisibleRef; 
    }

    
</script>
<template>    
    <li class="border-3  border-bottom" >
        <h2 class="p-1 text-center fw-bold">{{customer.name}}</h2> 
        <button type="button"
                class="btn btn-primary btn-sm"
                @click="fnToggleDetail()">
                상세정보 {{isDetailVisibleRef ? '숨기기' : '보기'}}
        </button>
        <ul class="list-unstyled" v-if="isDetailVisibleRef">
            <li><strong>전화번호:</strong>{{customer.phone}}</li>
            <li><strong>이메일:</strong>{{customer.email}}</li>
            <li><strong>주소:</strong>{{customer.address}}</li>
        </ul>
    </li>  
</template>
```
{% endcode %}

* 5 line : defineProps함수를 사용해서 isDetailVisible, customer 값을 받을 수 있게 선언 합니다
  * sDetailVisible, customer 에 대한 값 Validation를 하기 위해서 type, required, default 값 선언 합니다.
  * customer 는 Object로 필요한 것 만 Validation할 수 있습니다.
  * defineProps에서 정의한 변수는 vue의 특성한 단방향으로 수정을 하는 코드가 작성 되어 있으면 에러가 발생합니다.
* 20 line : isDetailVisible값은 컴포넌트 내부에서 변경 가능 하므로 ref 변수를 선언 해서 사용합니다. (isDetailVisibleRef )&#x20;

<figure><img src="../../../.gitbook/assets/image (6).png" alt="" width="563"><figcaption></figcaption></figure>

* 상세정보 숨기기 버튼을 클릭하면 해당 데이터만 숨겨지는 것을 확인 할 수 있습니다.

<figure><img src="../../../.gitbook/assets/image (240).png" alt="" width="563"><figcaption></figcaption></figure>

## 3. 자식에서 부모로 전달 ( emit )

Vue에서 데이터 전달은 단방향으로 전달 되므로 부모-자식간의 전달은 props를 통해서 전달을 합니다. 반대로 자식->부모간의 전달은 emit를 통해서 가능 합니다. emit 사용시 다음과 같은 주의사항이 있습니다.

* 데이터를 받고자 하는 곳에서 emit 명 생성
  *   자식 컴포넌트를 만들때 @emit이벤트명으로 선언&#x20;

      {% code lineNumbers="true" %}
      ```html
      const isDetailVisible = ref(true);

      const fnToggleDetailVisible = function() {
           isDetailVisible.value = !isDetailVisible.value;
      }

       
      <component-re-use                           
         :isDetailVisible = isDetailVisible
         v-for="customer in customers"
         :key = "customer.id"
         :customer= customer 
         @toggle-detail-visible="fnToggleDetailVisible"></component-re-use>  

      ```
      {% endcode %}



      * 3 line : emit을 통해서 이벤트를 받았을 떄 실행하는 함수
      * 13 line : emit을 통해서 이벤트를 받았을 떄 이벤트명 및 CallBack 함수 선언 \

* 데이터를 보내는 곳에서 받을 곳에 선언한 명을 사용해서 전달&#x20;
  *   자식 컴포넌트에서 emit 이벤트 생성성

      {% code lineNumbers="true" %}
      ```html
      // 자식 -> 부모로 전달 
      // const { emit } = getCurrentInstance();
      const emit = defineEmits(['toggle-detail-visible'])

      const fnEmit = ()=> {
         emit('toggle-detail-visible'); 
      } 

       
       
      <button type="button"
         class="btn btn-primary btn-sm"
         @click="fnEmit()">
         전체감추기 {{isDetailVisibleRef ? '숨기기' : '보기'}}
      </button> 
      ```
      {% endcode %}



      * 3 line : emit 이벤트 명 선언 : 보내야 할 곳에 정의된 받을 이벤트 명
        * 2 line : defineEmits 함수를 사용하지 않고 getCurrentInstance() 를 통해서 선언할 수 있습니다. ( setup() 함수의 첫번째 인자로 content 을 의미함 )
      * 5 line : emit 함수를 통해서 이벤트 전달&#x20;
        *   callback 으로 정의된 함수에 따라 파라메터로 데이터를 전달 할 수 있습니다.\


            ```javascript
            const fnToggleDetailVisible = function(name) {     
                ..... 
            }


            emit('toggle-detail-visible', '홍길동'); 
            ```



      * 13 line : emit를 호출 하기 위한 함수&#x20;

### 3-1. props 로 전달 받은 객체 변경&#x20;

props 로 통해서 받은 객체에 변경을 할려고 하면 오류가 발생 합니다. 원인은 vue 구조상 단방향으로 데이터를 전달하기 때문입니다. 만약 변경이 필요하면 별도의 반응형 객체를 만들어서 사용해야 합니다.

```javascript
 const isDetailVisibleRef = ref(props.isDetailVisible); 

 const fnToggleDetail = function() {   
    isDetailVisibleRef.value = !isDetailVisibleRef.value;  
 } 
```

### 3-2. emit통해서rops 로 전달 받은 객체 변경

props 받은 객체를 반응형 변수에 직접 넣을 수 없어서 watch를 이용해서 우회 작업이 가능 합니다.

<pre class="language-javascript" data-line-numbers><code class="lang-javascript"><strong>watch(props, (newValue, oldValue) => { 
</strong>    fnChangeDetailVisibleRef(newValue.isDetailVisible);
})

const fnChangeDetailVisibleRef = (value)=> {   
    isDetailVisibleRef.value = value
};
</code></pre>

* 1 line : watch의 첫번째 파라메터에 감시하고자하는 객체를 넣으며 vue는 해당 객체의 변경 사할이 있음면 이벤트를 발행 합나다. 이것으로 이용해서 변경이 되었을떄 작업할 함수를 호출 합니다.
* 5 line : watch에서 실행한 함수로 여기에서 반응형 객체의 값을 변경 합니다. 그럼 반응형 객체를 사용하고 있는 곳은 변경되게 됩니다.

아래는 props, emit, watch를 사용한 소스 코드 입니다.

<details>

<summary>부모 소스 </summary>

```html
<script setup>
  import { ref } from 'vue';
  import Component_00 from '@/components/rtComponent/Component_00.vue';
  import Component_01 from '@/components/rtComponent/Component_01.vue';
  import ComponentReUse from '@/components/rtComponent/ComponentReUse.vue';

  const customers = ref([
    { 
      id : 1, 
      name: '홍길동',
      phone: '010 9321 1234',
      email: 'hong01@gmail.com',
      address: '서울 송파구 잠실동'
    },
    { 
      id : 2, 
      name: '김바둑',
      phone: '010 7239 2345',
      email: 'kim01@gmail.com',
      address: '서울 송파구 삼전동'
    }
  ]);

 

  const isDetailVisible = ref(true);

  const fnToggleDetailVisible = function() {
      isDetailVisible.value = !isDetailVisible.value;
  }
```

</details>

<details>

<summary>자식 소스</summary>

```html
script setup>
    import {  ref , defineProps , defineEmits    } from 'vue';
    import {  watch, getCurrentInstance  } from 'vue';

    // props 불변 이다,. vue는 단방향이다.
    const props = defineProps({                    
        isDetailVisible: {
            type: Boolean,   
            required: true, 
            default: false
        },      
        customer: {
            id: {
                type: Number,
                required: true
            } 
        }
    }) 

   

    const isDetailVisibleRef = ref(props.isDetailVisible); 

    const fnToggleDetail = function() {   
       isDetailVisibleRef.value = !isDetailVisibleRef.value;  
    } 

    const fnChangeDetailVisibleRef = (value)=> {   
        isDetailVisibleRef.value = value
    }; 
   

    // 자식 -> 부모로 전달 
    // const { emit } = getCurrentInstance();
    const emit = defineEmits(['toggle-detail-visible'])

    const fnEmit = ()=> {
        emit('toggle-detail-visible'); 
    }

    watch(props, (newValue, oldValue) => { 
        fnChangeDetailVisibleRef(newValue.isDetailVisible);
    })
        

</script>
<template>    
    <li class="border-3  border-bottom" >
        <h2 class="p-1 text-center fw-bold">{{customer.name}}</h2> 
        <button type="button"
                class="btn btn-primary btn-sm"
                @click="fnToggleDetail()">
                상세정보 {{isDetailVisibleRef ? '숨기기' : '보기'}}
        </button>
        <button type="button"
                class="btn btn-primary btn-sm"
                @click="fnEmit()">
                전체감추기 {{isDetailVisibleRef ? '숨기기' : '보기'}}
        </button> 
        <ul class="list-unstyled" v-if="isDetailVisibleRef">
            <li><strong>전화번호:</strong>{{customer.phone}}</li>
            <li><strong>이메일:</strong>{{customer.email}}</li>
            <li><strong>주소:</strong>{{customer.address}}</li>
        </ul> 
    </li>  
</template>
```

</details>

| 부모 컴포넌트                                                                          | 자식 컴포넌                                                                           |
| -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| <img src="../../../.gitbook/assets/image (241).png" alt="" data-size="original"> | <img src="../../../.gitbook/assets/image (243).png" alt="" data-size="original"> |
