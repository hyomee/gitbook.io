# 리스트 렌터링

데이터 목록을 반복하고 목록의 각 항목에 대한 요소를 렌더링하여 애플리케이션의 흐름을 제어할 수 있다.

## 1. v-for 지시문 :key 속성

v-for 뒤에 v-bind:key(또는 :key)를 사용하여 key 속성에 바인딩하여 각 요소에 고유한 ID를 부여할 수 있다.

* &#x20;Vue의 가상 DOM 알고리즘은 새 DOM 트리를 이전 DOM 트리와 비교할 때 고유 ID가 있는 노드를 식별할 수 하므로 key 속성을 지정 해야 한다.
  * 어떤 아이템이 바뀌었는지 확인 하기 위해 명확히 식별할 수 있는 값이 필요한데 그것이 key 이다.&#x20;
  * `key`를 기준으로 렌더링 대상을 식별하면 부하가 적다.
    * key가 없으면 그냥 순서를 기반으로 짐작해서 렌더링 대상을 정하므로.더 많은 DOM을 업데이트가 발생한다.순서를 기반으로 짐작해서 렌더링 대상을 정합니다. 더 많은 DOM을 업데이트해야 하는 것이죠.

```html
<script setup>
    const nameItems = [  '홍길동', '김박수'  ]; 
    const nations = [  
        { id:1 , name : '한국'},
        { id:2 , name : '미국'} 
    ]; 
</script>


<template>
    <select>
        <option v-for="nation in nations" 
                :key="nation.id"
                v-bind:value="nation.id">
            {{ nation.name }}
        </option>
    </select> 
</template> 
```

| HTML                                                                             | Data                                                                             |
| -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| <img src="../../../.gitbook/assets/image (185).png" alt="" data-size="original"> | <img src="../../../.gitbook/assets/image (186).png" alt="" data-size="original"> |

배얄도 동일 함

```html
<li v-for="(name, x) in nameItems">
    {{ x }} :: {{ name }}
</li>
```

## 2. v-for 요소의 인덱스를 얻는 방법

v-for 에서 목록의 인텍스를 얻기 위해서는 item 영역의 두번째 인자를 사용하면 된다.

```markup
<element v-for="(alias, index) in list">
  {{ index }} - {{ alias }}
</element>
```

{% code lineNumbers="true" %}
```html
<script setup>
    const nations = [  
        { id:1 , name : '한국'},
        { id:2 , name : '미국'} 
    ]; 
</script>

<template>
    <div>
        <select>
            <option v-for="(nation, index) in nations" 
                    :key="nation.id"
                    v-bind:value="nation.id">
                    {{ index }} :: {{ nation.name }}
            </option>
        </select> 
    </div>
</template>
```
{% endcode %}

* 14 line : v-for 구문에서 item에 해당 하는 부분에 두번째 인자를 사용한다.
  * v-for="(nation, index) in nations 에서 index 이다. ( 인자 이름은 마음대로 .. )

| HTML                                                                             | Data                                                                             |
| -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| <img src="../../../.gitbook/assets/image (187).png" alt="" data-size="original"> | <img src="../../../.gitbook/assets/image (188).png" alt="" data-size="original"> |

배열도 동일함&#x20;

```html
<li v-for="(name, x) in nameItems" :key="name">
    {{ x }} :: {{ name }}
</li>
```

## 3. 중첩 배열&#x20;

일반 프로그램과 동일하게 중첩 v-for를 사용한다.

{% code lineNumbers="true" %}
```html
<script setup>        
    const customers = [  
        { 
            id:1 , 
            name : '홀길동',
            addresss : [
                { id: 1, addressdiv: '기본',fuladdress: '서울 송파'},
                { id: 2, addressdiv: '기타',fuladdress: '서울 강동'}
            ]
        },
        { 
            id:2 , 
            name : '김말자',
            addresss : [
                { id: 3, addressdiv: '기본',fuladdress: '강원 춘천'},
                { id: 4, addressdiv: '기타',fuladdress: '강원 영월'}
            ]
        },
    ]; 
</script>

<template>
    <div>
        <table > 
            <tr> 
                <td> 중첩 배열 </td> 
                <td width="200">
                    <p v-for="customer in customers" :key="customer.id"> 
                        <p v-for="addresss in customer.addresss" :key="addresss.id">
                            {{ customer.name  }} : {{ addresss.addressdiv }} , 
                            {{ addresss.fuladdress }}                       
                        </p>
                    </p>
                </td>
            </tr>
        </table>
    </div>
</template>
```
{% endcode %}

* 28 \~ 32 line : 중첩 v-for 사용

| HTML                                                                             | Data                                                                             |
| -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| <img src="../../../.gitbook/assets/image (189).png" alt="" data-size="original"> | <img src="../../../.gitbook/assets/image (191).png" alt="" data-size="original"> |

## 4. v-for, v-if 사용시 Computed 속성 사용

코드 작성시 for 반볷문안에 if문을 사용해서 특정 데이터를 필터 하는 경우가 있다.

### 4-1. template 에서 v-if 사용

아래 코드는 v-for , v-if를 사용해서 작성한 코드 이다. &#x20;

```html
<script>
    export default {
        data() {
            return {
                users: [
                    { name: '홍일이', age: 33 },
                    { name: '홍이이', age: 33 },
                    { name: '홍삼이', age: 17 },
                    { name: '홍사이', age: 16 }
                ]
            }
        }
    }
</script>

<template>
    <template v-for="user in users" :key="user.name">
      <p v-if="user.age >= 18">{{ user.name }}</p>
    </template>
</template>
  
```

### 4-1. Computed 속성 사용

### 4-1-1. Option API

```html
<script>
    export default {
        data() {
            return {
                users: [
                { name: '홍일이', age: 33 },
                { name: '홍이이', age: 33 },
                { name: '홍삼이', age: 17 },
                { name: '홍사이', age: 16 }
                ]
            }
        },
        computed: {
            legalUsers() { 
                return this.users.filter(user => user.age >= 18);
            }
        }
    }
</script>

<template>
    <template v-for="user in legalUsers" :key="user.name">
      <p v-if="user.age >= 18">{{ user.name }}</p>
    </template>
</template>
```

### 4-1-2. Composition API

```html
<script setup>
    const users = [
                { name: '하나', age: 33 },
                { name: '둘', age: 33 },
                { name: '셋', age: 17 },
                { name: '넷', age: 16 }
                ] ;

    const fnLegalUsers = () => users.filter(user => user.age >= 18); 
 
</script>

<template>
    <template v-for="user in fnLegalUsers()" :key="user.name">
      <p v-if="user.age >= 18">{{ user.name }}</p>
    </template>
</template>
```

```html
<script>
    import {computed} from 'vue';

    const users = [
                    { name: '하나', age: 33 },
                    { name: '둘', age: 33 },
                    { name: '셋', age: 17 },
                    { name: '넷', age: 16 }
                ];

    const fnUsers = () => {
        const fnLegalUsersCompted = computed(() => { 
                console.log("setup() : fnLegalUsersCompted",  users);
                return  users.filter(user => user.age >= 18)
            }); 

        const fnLegalUsers = () => { 
            console.log("setup() : fnLegalUsers",  users);
            return users.filter(user => user.age >= 18)
        }; 

        return {
            fnLegalUsersCompted, 
            fnLegalUsers
        }
    }

    export default {
        data() {
            return {
                users
            }
        },  
        setup() { 
            const {fnLegalUsersCompted, fnLegalUsers } = fnUsers();
            return {
                fnLegalUsersCompted,
                fnLegalUsers,
            }
        }
         
    }
</script> 

 
<template>
    <template v-for="user in fnLegalUsersCompted" :key="user.name"> 
        {{ user.name }} <br/>
    </template>
</template>

// or 

<template>
    <template v-for="user in fnLegalUsers ()" :key="user.name"> 
        {{ user.name }} <br/>
    </template>
</template>
```
