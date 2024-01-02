# 빌드 인디렉티브 (Built-in Directives)

<figure><img src="../../../.gitbook/assets/image (173).png" alt=""><figcaption><p>디렉티브 문법을 시각화</p></figcaption></figure>

## 1. 데이터 바인딩

데이터 바인딩은 구성 요소의 View(_템플릿_)와 Logic(_스크립트_) 간의 통신을 하는 것으로 세가지 유형의 바인딩이 있다.

* 단방향 데이터 바인딩
  * script to template : _String Interpolation_ 또는 _Attribute binding (_빌드 인디렉티브_)_
  * template to script : 이벤트 전달&#x20;
* 양방향 데이터 바인딩
  * script to template , template to script 동시에 적용

### 1-1. v-bind

컴포넌트에 선언된 변수와 HTML 태그 속성을 결합하는 것으로 단방향 바인딩로 테이터가 변경이 되지 않는다.

```html
<template>
    <input type="text" v-bind:value="vbindMsg" />
    <div> {{ vbindMsg }}</div>
</template>

<script setup>
    const vbindMsg = ref("v-bind");
</script>
```

<figure><img src="../../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

### 1-2. v-html

이중 중괄호 바인딩은 html 을 일반 텍스트로 해석하므로 HTML을 출력하기 위해서는 v-html 디렉티브를 사용해야 한다.

* HTML을 동적으로 렌더링하는 것은 XSS 공격에 취약하다.
* SFC에서 style scoped지정시 HTML로 변환된 것은 적용 받지 않는다.

```html
<template>
  <p v-html="msg"></p>
</template>

<script>
export default {
  data() {
    return {
      msg: '<strong>Hello Vue</strong>'
    }
  }
}
</script>
```

### 1-3. v-pre

해당 엘리먼트를 포함한 모든 자식 엘리먼트들의 값을 컴파일 하지 않으므로 그대로 출력이 됩니다.

```html
<template> 
  <div>{{ msg }}</div>              // 결과 : 일반
  <div v-pre>{{ propMsg }}</div>    // 결과 : {{ propMsg }}
</template>
```

```javascript
const preMsg = ref("pre디렉티브");
const msg = ref("일반");
```

### 1-4. v-text

엘리먼트의 텍스트 속성 설정으로 해당 tag의 값을 변경 한다. 일부를 변경을 하는 경우 이중 중괼호를 사용해야 한다.

```html
<span v-text = "vtextMsg"></span>
<span>일부를 변경 {{vtextMsg}}입니다.</span>
```

```
<template>
    <div>
        <h3>=== v-text ===</h3>
        <span v-text = "vtextMsg"></span>
    </div>
</template
```

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption><p>vue devtools</p></figcaption></figure>

```html
// 1. Composition API - script setup
<script setup>
    const vtextMsg = " 텍스트 속성 설정으로 해당 tag의 값을 변경";
</script>

// 2. Composition API - script()
<script>
    export default {
        setup() {
            const vtextMsg = " 텍스트 속성 설정으로 해당 tag의 값을 변경";
            return {
                vtextMsg
            }
        }   
    }
</script>

// 3. Option API
<script>
    export default {
        data() { 
            return {
                vtextMsg: '텍스트 속성 설정으로 해당 tag의 값을 변경'
            }
        }  
    }
</script>

// 4. Option API & Composition API
// 특별한 경우가 아니면 사용하지 말아야 함
<script>
    export default {
        data() { 
            return {
                vtextMsg: ''
            }
        },
        setup() {
            const vtextMsgSetup = " 텍스트 속성 설정으로 해당 tag의 값을 변경";

            return {
                vtextMsgSetup
            }
        }, 

        mounted() { 
            this.vtextMsg = `${this.vtextMsgSetup} data 정의`;
        } 

    }
</script>
 

```

## 2. 양방향 바인딩

### 2-1. v-model

컴포넌트에 선언된 변수와 HTML 태그 속성을 결합하는 것으로 양방향 바인딩으로 테이터가 변경된다.

```html
<input type="text" v-model="vmodelMsg" />
<div> 양방형 : {{ vmodelMsg }}</div>
```

```javascript
const vmodelMsg = ref("v-model");
```

<figure><img src="../../../.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

## 3. 조건부  렌더링

응용 프로그램을 빌드할 때 특정 조건에 따라 HTML 요소를 표시하거나 숨겨야 하는 경우 사용한다.

* 조건부 렌더링 지시문은 true 또는 false를 값으로 반환하는 Javascript 식을 허용한다.&#x20;
* 그런 다음 Vue는 평가 출력에 따라 요소를 렌더링하거나 렌더링하지 않는다.

### 3-1. v-show

* 요소를 숨기거나 표시하는 데 사용된다.&#x20;
* 표현식을 평가하고 해당 결과에 따라 요소의 `display` CSS 속성을 토글한다.
* 초기 렌더링 때 항상 요소를 렌더링하고, 그 후에 표현식을 평가하여 요소를 표시하거나 숨긴다.

```html
<script setup>
    const isShow = true;
</script>

<template>
    <h1 v-show="isShow">showTest</h1>
</template>
```

| isShow = true                                                                  | isShow = false                                                                     |
| ------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------- |
| <img src="../../../.gitbook/assets/image (2).png" alt="" data-size="original"> | <img src="../../../.gitbook/assets/image (1) (1).png" alt="" data-size="original"> |

### 3-2. v-if, v-else-if, v-else

* 요소가 DOM에 렌더링 되거나 제거되는지 여부를 조건부로 결정한다.
  * 즉,  표현식을 평가하고 해당 결과에 따라 요소를 렌더링하거나 제거한다.
  * true : 요소가 DOM에서 제거
  * 요소의 존재 여부를 동적으로 제어할 수 있어 필요에 따라 특정 조건에서만 요소를 렌더링 할 수 있다.
* 초기 렌더링 때 표현식을 평가하여 요소를 렌더링 하거나 제거한다.

```html
<script setup>
    import { ref } from 'vue'
    const isShow = true;
    const isIf = ref('A');
    const isToggle = ref(true);
</script>

<template>
    <h1 v-show="isShow">showTest</h1>    
    <button @click="isToggle = !isToggle">전환</button>
    <label v-if="isToggle">조건 : {{  isToggle}}</label>
    <label v-else>조건 : {{  isToggle }}</label><br/>

    <input type="text"
           v-bind:value="isIf"
           @input="isIf = $event.target.value" /> {{  isIf }}
    <label v-if="isIf=='A'">조건 A</label>
    <label v-else-if="isIf=='B'">조건 B</label>
    <label v-else>조건 틀림</label>
</template>

```

<figure><img src="../../../.gitbook/assets/image (3).png" alt="" width="518"><figcaption><p>초기 웹 페이지 화면</p></figcaption></figure>

전환 버튼을 클릭 하고 텍스트 박스에 B를 입력 하면 변경 되는 것을 확인 할 수 있다.

<figure><img src="../../../.gitbook/assets/image (4).png" alt="" width="515"><figcaption><p>화면에서 조작 후 </p></figcaption></figure>

### 3-3. `v-if` vs `v-show`

* v-if
  * "실제" 조건부 렌더링으로 조건부 블록이 전환될 경우, 블록 내 이벤트 리스너와 자식 컴포넌트가 제대로 제거되거나 재생성한다.
  * &#x20;초기 조건이 false면 아무 작업도 수행하지 않고  true가 될 때까지 렌더링되지 않는다.
  * &#x20;전환 비용이 더 높다
  * 실행 중에 조건이 변경되지 않는 경우 `v-if`를 사용
* v-show
  * 엘리먼트는 CSS 기반으로 전환 되므로, 초기 조건과 관계없이 항상 렌더링된다.
  * 초기 렌더링 비용이 더 높다
  * 매우 자주 전환해야 하는 경우 `v-show 사용`

## 4. 리스트  렌더링

### 4-1. v-for&#x20;

소스 데이터를 기반으로 엘리먼트 또는 템플릿 블록을 여러 번 렌더링 한다.

* **적용 되는 값: `Array | Object | number | string | Iterable`**
* &#x20;item`in items` 형식의 특별한 문법이 적용된다.
  * `tems`: 배열이고, `item`(이하 아이템)은 배열 내 반복되는 앨리먼트의 \*\*별칭(alias)\*\*이다.

```markup
<element v-for="item in list">
  {{ item }}
</element>
```

{% code lineNumbers="true" %}
```html
<script setup>
    const nameItems = [  '홍길동', '김박수'  ]; 
    const nations = [  
        { name : '한국'},
        { name : '미국'} 
    ]; 
</script>

<template>
    <div>
        <table >
            <tr>
                <td rowspan="2" width="120">  v-for : </td>
                <td width="120"> 이름 </td> 
                <td width="120">
                    <li v-for="name in nameItems">
                        {{ name }}
                    </li>
                </td>
            </tr>
            <tr> 
                <td> 국적 </td> 
                <td>
                    <select>
                        <option v-for="nation in nations">
                            {{ nation.name }}
                        </option>
                    </select> 
                </td>
            </tr>
        </table>
    </div>
</template>
```
{% endcode %}

* 2 line : nameItems 변수에 배열로 이름 할당&#x20;
  * 16 line : li 태그에 v-for 를 사용하여 nameItems 변수에 있는 값을 화면 출력한다.
* 3 line : nations 변수에 나라를 객체로 할당
  * 25 line : select-option 태그에 v-for 를 사용하여 nations 변수에 있는 값을 화면 출력한다.

<figure><img src="../../../.gitbook/assets/image (184).png" alt=""><figcaption></figcaption></figure>

## 5. 이벤트 핸드링 (v-on)

v-on 디렉티브는 단축 문법으로 `@` 기호를 사용하며,  DOM 이벤트를 수신하고 트리거될 때, 사전 정의해둔 JavaScript 코드를 실행할 수 있다. `v-on:click="handler"` 또는 줄여서 `@click="handler"`와 같이 사용하며 핸드러 값은 다음 중 하나일 수 있다

1. **인라인 핸들러:** 이벤트가 트리거될 때 실행되는 인라인 JavaScript(네이티브 `onclick` 속성과 유사).
2. **메서드 핸들러:** 컴포넌트에 정의된 메서드 이름 또는 메서드를 가리키는 경로

### 5-1. Javascript 이벤트 객체

v-on:click="handler" 또는 줄여서 @click="handler"은 실제 클릭 이벤트에 대한 것으로 자바스트립트의 이벤트 객체를 사용하는 것한다.

참고 :  [mdn Event](https://developer.mozilla.org/en-US/docs/Web/API/Event), [HTML attribute reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes#event\_handler\_attributes)

### 5-1. [html event handler](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes#event\_handler\_attributes)

html event handler 는 v-on:속성에 on를 제거 하고 작성한다. 단축은 @를 붙인다.&#x20;

| Event handler | v-on       | v-on 축약 |
| ------------- | ---------- | ------- |
| onclick       | v-on:click | @click  |
| onkeyup       | v-on:keyup | @keyup  |
| oninput       | v-on:input | @input  |

* The [event handler](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes#event\_handler\_attributes) attributes: `onabort`, `onautocomplete`, `onautocompleteerror`, `onblur`, `oncancel`, `oncanplay`, `oncanplaythrough`, `onchange`, `onclick`, `onclose`, `oncontextmenu`, `oncuechange`, `ondblclick`, `ondrag`, `ondragend`, `ondragenter`, `ondragleave`, `ondragover`, `ondragstart`, `ondrop`, `ondurationchange`, `onemptied`, `onended`, `onerror`, `onfocus`, `oninput`, `oninvalid`, `onkeydown`, `onkeypress`, `onkeyup`, `onload`, `onloadeddata`, `onloadedmetadata`, `onloadstart`, `onmousedown`, `onmouseenter`, `onmouseleave`, `onmousemove`, `onmouseout`, `onmouseover`, `onmouseup`, `onmousewheel`, `onpause`, `onplay`, `onplaying`, `onprogress`, `onratechange`, `onreset`, `onresize`, `onscroll`, `onseeked`, `onseeking`, `onselect`, `onshow`, `onsort`, `onstalled`, `onsubmit`, `onsuspend`, `ontimeupdate`, `ontoggle`, `onvolumechange`, `onwaiting`.

### 5-1. 수식어

#### 5-1-1. 이벤트 수식어

@click.prevent.once 형태로 여러개 사용가능하다.

<table><thead><tr><th width="151"></th><th></th></tr></thead><tbody><tr><td>.stop</td><td><p>이벤트.전파방지</p><p>e.stopPropagation()</p></td></tr><tr><td>.prevent</td><td><p>브라우저의 기본 동작 금지</p><p>e.preventDefault()<br><br>// submit 이벤트가 더 이상 페이지 리로드하지 않습니다.<br></p></td></tr><tr><td>.capture</td><td>이벤트 리스너의 capture옵션 활성시키다.</td></tr><tr><td>.self</td><td> 이벤트가 자식 엘리먼트가 아닌 현재 엘리먼트에서 발생했을 때만 핸들러를 호출한다.</td></tr><tr><td>.once</td><td>클릭이벤트는 여러번 가능하지만 handler라는 메서드는 한번만 실행됨</td></tr><tr><td>.passive</td><td>이벤트리스너의 passive 옵션 활성시키다.</td></tr><tr><td>.exact</td><td>정확히 해당 키만 눌렀을 때 핸들러 호출한다.<br>@click.ctrl.exact</td></tr><tr><td>.left</td><td>마우스 왼쪽 버튼을 눌렀을 때 핸들러 호출한다.</td></tr><tr><td>.right</td><td>마우스  오른쪽 버튼을 눌렀을 때 핸들러 호출한다.</td></tr><tr><td>.middle</td><td>마우스가운데 버튼을 눌렀을 때 핸들러 호출한다.</td></tr></tbody></table>

```markup
<!-- 클릭 이벤트 전파가 중지. -->
<a @click.stop="doThis"></a>

<!-- submit 이벤트가 더 이상 페이지 리로드하지 않음. -->
<form @submit.prevent="onSubmit"></form>

<!-- 수식어를 연결할 수 있습니다. -->
<a @click.stop.prevent="doThat"></a>

<!-- 이벤트에 핸들러 없이 수식어만 사용할 수 있다. -->
<form @submit.prevent></form>

<!-- event.target이 엘리먼트 자신일 경우에만 핸들러가 실행된다. -->
<!-- 예를 들어 자식 엘리먼트에서 클릭 액션이 있으면 핸들러가 실행되지 않는다. -->
<div @click.self="doThat">...</div>

<!-- 이벤트 리스너를 추가할 때 캡처 모드 사용 -->
<!-- 내부 엘리먼트에서 클릭 이벤트 핸들러가 실행되기 전에, 여기에서 먼저 핸들러가 실행된다. -->
<div @click.capture="doThis">...</div>

<!-- 클릭 이벤트는 단 한 번만 실행된다. -->
<a @click.once="doThis"></a>

<!-- 핸들러 내 `event.preventDefault()`가 포함되었더라도 -->
<!-- 스크롤 이벤트의 기본 동작(스크롤)이 발생한다.        -->
<div @scroll.passive="onScroll">...</div>
```

#### 5-1-2. 키 수식어

키보드와 관련이 있는 수식어

**입력키 별칭** : Vue는 가장 일반적으로 사용되는 키에 대한 별칭을 제공

* `.enter`
* `.tab`
* `.delete` ("Delete" 및 "Backspace" 키 모두 캡처)
* `.esc`
* `.space`
* `.up`
* `.down`
* `.left`
* `.right`

**시스템 입력키 수식어 :** 마우스 또는 키보드 이벤트 리스너는 아래 수식어를 사용하여, 해당 입력키를 누를 때만 트리거 되도록 할 수 있다:

* `.ctrl`
* `.alt`
* `.shift`
* `.meta`

```html
<!-- `key`가 `Enter`일 때만 `submit`을 호출다 -->
<input @keyup.enter="submit" />

// KeyboardEvent.key를 통해 유효한 입력키 이름을 kebab-case로 변환하여 수식어로 사용할 수 있다:
<input @keyup.page-down="onPageDown" />

<!-- Alt + Enter -->
<input @keyup.alt.enter="clear" />

<!-- Ctrl + Click -->
<div @click.ctrl="doSomething">시작하기</div>
```

참고 : [이벤트 핸들링](https://ko.vuejs.org/guide/essentials/event-handling.html)

## 6. 한번만  렌더링 (v-once)

요소를 처음 렌더링한 후 Vue는 이를 정적 콘텐츠로 처리하고 다시 렌더링하지 않는다.

```html
<p v-once>초기값 cnt : {{ cnt }}</p>
```

버튼을 클릭 해도 초기값은 변경되지 않는다.

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## 7. 메모 (v-memo)

## 8. 템플릿 숨기기(v-cloak)

## 9. Slot (v-slot) &#x20;
