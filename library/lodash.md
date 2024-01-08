# Lodash

**사이트 :** [**https://lodash.com/**](https://lodash.com/)

Lodash는 배열, 숫자, 객체, 문자열 등의 작업을 JavaScript를 더 쉽게 만들어  주는 라이브로리로  다음과 같은 경우에 적합하다.

* 배열, 객체, 문자열 반복하기
* 값 조작 및 테스트
* 복합 함수 만들기

API 설명 : [https://lodash.com/docs/4.17.15](https://lodash.com/docs/4.17.15)

{% code lineNumbers="true" %}
```html
<script setup>
    import {computed } from 'vue';
    import { sortBy  } from 'lodash';

    const users = [
                { name: '하나', age: 33 },
                { name: '둘', age: 33 },
                { name: '셋', age: 17 },
                { name: '넷', age: 16 }
                ] ;

    const fnLegalUsersComputed = computed(() => { 
        return users.filter(user => user.age >= 18)
    }); 

    const fnLegalUsers = () => { 
        console.log(users);
        return users.filter(user => user.age >= 18)
    }; 

    // Lodash 사용 
    const fnSortUsers = () => {  
        return sortBy(users, ['age'])
    };
 
</script>
 
<template>
    <template v-for="user in fnSortUsers()" :key="user.name"> 
        {{ user.name }} <br/>
    </template>
</template>  

```
{% endcode %}

<figure><img src="../.gitbook/assets/image (218).png" alt=""><figcaption></figcaption></figure>

## Lodash 와 ES6 차이점

<table><thead><tr><th width="116">항목</th><th width="278">장점</th><th>단점</th></tr></thead><tbody><tr><td>Lodash</td><td>메소드에서 제공하는 다양한 기능으로 생산성이 높다.</td><td>외부 라이브러리를 사용하기 때문에 무겁다.<br>ES6에 비하여 속도가 느리다</td></tr><tr><td>ES6</td><td>속도가 빠르다.<br> 호환성이 좋다.</td><td>예외 처리를 잘 해야한다.<br>Array 타입밖에 지원을 안하기 때문이다. 복잡한 데이터 처리에는 직접 메소드를 만들어야 하므로 번거롭다</td></tr></tbody></table>



