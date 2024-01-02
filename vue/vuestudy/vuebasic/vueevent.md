# Methods & 이벤트처리

## 1. Methods

Option API에서 컴포넌트 인스턴스에서 사용할 메서드를 선언한다,

* 선언된 메서드는 컴포넌트 인스턴스에서 직접 접근하거나, 템플릿 표현식에서 사용할 수 있댜.&#x20;
* 모든 메서드에는 해당 컴포넌트 인스턴스에 자동으로 바인딩된 `this` 컨텍스트가 있으며, 주위 컴포넌트로 전달된 경우에도 유지된다.
  * this 로 참조 해야 한다
* 메서드를 선언할 때 화살표 함수를 사용하지 않도록 해야 하는데, `this`를 통해 컴포넌트 인스턴스에 접근할 수 없기 때문이다.

{% code lineNumbers="true" %}
```html
<script>
    export default {
        data() {
            return { cnt: 1 }
        },
        methods: {
            plus() {
                this.cnt++
            }
        },
        created() {
            this.plus() 
        }
    }
</script>

<template>
    <h1>메서드</h1>
    <p>
        <button @click="plus();">plus 클릭</button>{{ cnt }}
    </p>
</template>
```
{% endcode %}

* 4 line : cnt 변수 선언 및 할당 하였다
* 6 line :  컴포넌트 내부에서 사용할 메서드 선언 하였다
  * 7 line : plus 함수 선언 하였다.
    * script 내부에서 this. 으로 접근해야 한다.
    * template 에서는 this. 없이  함수로 접근해야 한다.

<figure><img src="../../../.gitbook/assets/image (193).png" alt="" width="563"><figcaption></figcaption></figure>
