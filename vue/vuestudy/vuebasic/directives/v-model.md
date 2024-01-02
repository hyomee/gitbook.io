---
description: v-model or v-bin @input 적용
---

# 양방향 바인딩 v-model

<mark style="color:green;">**v-model양식 입력을 참조 또는 반응형 개체에 바인딩 간단한 2단계로 수행됩니다.**</mark>

1. <mark style="color:green;">**ref 를 만듭니다 :  const text = ref('').**</mark>
2. <mark style="color:green;">**속성에 참조를 할당 합니다 : v-model.\<input v-model="text" type="text" />**</mark>

\<input v-model:"ref" />와 같이 html의 input 태그와 결합되어 양방향 바인딩을 하는 디렉티브로 사용자가 입력을 하면 참조가 변경이 되고 반대로 프로그램에서 변경을 하면 입력 필드의 값이 변경이 됩니다.

```javascript
<script setup>
import { ref } from "vue";

const text = ref("알수없는값");
</script>
<template>
  <input v-model="text" type="input" />
  <div>{{ text }}</div>
</template>
```

1. const text = ref("알수없는값"); text에 초기 값 설정
2. v-model="text" : 양방향 바인딩을 위해 input tag에 추가

<figure><img src="../../../../.gitbook/assets/image (64).png" alt="" width="407"><figcaption><p> 양방향 바인딩 : v-model</p></figcaption></figure>

**text에 "알수없는값"으로 초기화 되고 입력 필드에 값을 수정하면 화면에 수정된 값이 표시 됩니다**.

만약 단방향 바인딩을 원한다면 다음과 같은 코드로 수정을 해야 합니다.

```javascript
<script setup>
import { ref } from 'vue'

const text= ref('알수없는값')
</script>
<template>
  <input :value="text" type="input" />
  <div>{{ text}}</div>
</template>
```

1. const text = ref("알수없는값"); text에 초기 값 설정
2. :value="text" or v-bind: 단방향 바인딩을 위해 input tag에 추가

<figure><img src="../../../../.gitbook/assets/image (65).png" alt="" width="288"><figcaption><p>단방향 바인딩 : v-text</p></figcaption></figure>

**text에 "알수없는값"으로 초기화 되고 입력 필드에 값을 수정하여도 입력 필드는 수정이 되지 않는다**.

## 1. v-bind 를 양방향으로 변경

v-bind: @input 이용해서 v-model와 같이 양방향 바인딩으로 할 수 있습니다,

```javascript
<script setup>
import { ref } from "vue";

const text = ref("알수없는값");
</script>
<template>
  <input v-bind:value="text"
   @input="text = $event.target.value" 
   type="input" />
  <div>{{ text }}</div>
</template>
```

1. const text = ref("알수없는값"); text에 초기 값 설정
2. :value="text" or v-bind: 단방향 바인딩을 위해 input tag에 추가
3. @input="text = $event.target.value" : 사용자 입력 필드가 변경이 되면 text에 업데이트 됩니다.

## 2. reactive()를 사용한 바인딩

```javascript
<script setup>
import { reactive } from 'vue'

const person = reactive({ firstName: 'John', lastName: 'Smith' })
</script>
<template>
  <input v-model="person.firstName" type="input" />
  <input v-model="person.lastName" type="input" />
  <div>Full name is {{ person.firstName }} {{ person.lastName }}</div>
</template>
```

1. import { reactive } from 'vue' : reactive 임포트
2. const person = reactive({ firstName: 'John', lastName: 'Smith' }) : person 객체에 저장
3. v-model="person.firstName" : 바인딩

<figure><img src="../../../../.gitbook/assets/image (63).png" alt="" width="316"><figcaption></figcaption></figure>

이름을 입력하면 바인딩되는 것을 확인 할 수 있습니다.

## 3. 예제

### 3-1. textarea

HTML input과 동일하게 적용됨

```
<textarea v-model="longText" />
```

```javascript
<template>
    <div>
        <h3>textarea 바인딩</h3> 
        <textarea v-model="longText" />
        <div>{{ longText }}</div>
    </div>
</template>

<script setup>
import { ref } from 'vue'

const longText = ref("텍스트 영역 바인딩...,")
</script>
```

### 3-2. select

드롭다운(선택)필드

```
<select v-model="country" >
```

선택한 value : \{{ country \}}에 선택한 행의 value를 표시 합니다.

```javascript
<template>
    <div> 
        <select v-model="country" >
            <option value="1">대한민국</option>
            <option value="2">영국</option>
            <option value="3">호주</option>
        </select>
        <div>선택한 value : {{ country }}</div>
    </div>
</template>

<script setup>
import { ref } from 'vue'

const country = ref('')
</script>
```

\<option>대한민국\</option>에서 value를 정의하지 않으며 텍스트("대한민국")을 country에 바인딩 합니다.

### 3-3. checkbox

#### 3-3-1. CheckBox는 멀티 선택&#x20;

1. const checkCountry = ref(\[]) : 참조 변수 checkCountry를 배열로 초기화 합니다.
2. v-model="checkCountry" value="korea" type="checkbox" : v-model 바인딩 선언합니다.

```javascript
<template>
    <div> 
        <label><input v-model="checkCountry" value="korea" type="checkbox" />대한민국</label>
        <label><input v-model="checkCountry" value="eng" type="checkbox" />영국</label>
        <div>선택한 checkCountry : {{ checkCountry }}</div>
    </div>

</template>

<script setup>
import { ref } from 'vue'

const checkCountry = ref([])
</script>
```

&#x20;선택한 checkCountry : \[ "korea", "eng" ]로 표시 됩니다.

#### 3-3-2. CheckBox 부울 값&#x20;

1. const checked = ref(true) : 참조 변수 checked 를 부울 선언
2. v-model="checked" type="checkbox" : v-model 바인딩 선언합니다.

```javascript
<template>
    <div> 
        <label><input v-model="checked"   type="checkbox" />대한민국</label> 
        <div>선택한 checked : {{ checked }}</div>
    </div>
</template>

<script setup>
import { ref } from 'vue'

const checked = ref(true)
</script>
```

대한민국 선택한 checked : true가 표시 됩니다.

true-value, false-valued을   사용해서  선택/선택취소 값을 사용자가 정의하여는작성하면  코드는 다음과 같습니다.

```javascript
<template>
    <div> 
        <label><input type="checkbox"
                      v-model="checked" 
                      true-value="Yes" 
                      false-value="No"/>대한민국</label> 
        <div>선택한 checked : {{ checked }}</div>
    </div>

</template>

<script setup>
import { ref } from 'vue'

const checked = ref()
</script>
```

선택한 checked : Yes 로 표시 됩니다.

### 3-4. radio

### 3-5. v-model option

#### 3-5-1. trim

#### 3-5-2. number

#### 3-5-3. lazy

