# 컴포지션 API: 생명주기 훅

> 컴포지션 API 에 대한 자세한 설명은 [vue 공식 사이트 컴포지션 AP](https://ko.vuejs.org/api/composition-api-lifecycle.html)I: 생명주기 훅을 참고 하세요.

## 1. 훅설명

컴포지션 API를 사용하기 위해서는 사용하고자 하는 훅를 import를 사용해서 참조해야 합니다.

```javascript
import { onBeforeMount, onMounted  } from 'vue'
import { onBeforeUpdate, onUpdated } from 'vue'
import { onBeforeUnmount,  onUnmounted } from 'vue'
```

<table><thead><tr><th width="200">컴포지션 API </th><th>설명</th></tr></thead><tbody><tr><td>setup</td><td>컴포넌트를 생성되면 호출되므로 data를 초기화 할 수 있다</td></tr><tr><td>onBeforeMount</td><td><p>render 함수를 호출 하기 전으로 renderTracked으로 관찰 할 수 있다.</p><pre class="language-javascript"><code class="lang-javascript">onBeforeMount(() => {
   console.log("onBeforeMount ....");   
});
</code></pre></td></tr><tr><td>onMounted</td><td><p>DOM 엘리먼트로 마운트 된 후 호출되므로 엘리먼트 참조가 가능한다. 즉 ref와 같은 함수를 통해서 작성한 변수를 참조 할 수 있다. onRenderTracked 훅 호출 된다, ( <strong>가상 DOM -> DOM 반영</strong> )<br><mark style="color:orange;">서버 사이드 렌더링 중에 호출 되지 않음.</mark></p><pre class="language-javascript"><code class="lang-javascript"><strong>onMounted(() => {
</strong>   console.log("onMounted....");   
});
</code></pre></td></tr><tr><td>onBeforeUpdate</td><td><p>데이터가 변경되어 DOM에 반영 되기 전 호출됨으로 엘리머크를참조하는수변 참조는 이전 값<br><br><mark style="color:orange;">* 서버 사이드 렌더링 중에 호출 되지 않음.</mark></p><pre class="language-javascript"><code class="lang-javascript">onBeforeUpdate(() => {
  console.log("onBeforeUpdate ....");   
});

</code></pre></td></tr><tr><td>onUpdated</td><td><p>데이터가 변경되어 DOM에 변경 완료되는 시점에 호출하므로 참조된 변수를 제어 할 수 있다. 단 자식 노드가 있으면 nextTick을 사용해서 자식 업데이트가 완료될 때 까지 기다려야 한다.</p><p><br>* <mark style="color:orange;">서버 사이드 렌더링 중에 호출 되지 않음.</mark><br><mark style="color:orange;">* updated 훅에서 컴포넌트 상태를 변경하면 안되는데, 무한 업데이트 루프가 발생할 수 있기 때문입니다</mark>!<br><br>update() {<br>   this.$nextTick(function() {<br>     // 모든 자식 노드 업데이트 됨<br>    })<br>}</p><pre class="language-javascript"><code class="lang-javascript">onUpdated(() => {
  console.log("onUpdated....");   
});
</code></pre></td></tr><tr><td>onBeforeUnmount</td><td><p>컴포넌트가 해제 되기 전에 호출되므로 모든 기능 사용 할 수 있다.<br><br><mark style="color:orange;">* 서버 사이드 렌더링 중에 호출 되지 않음.</mark></p><pre class="language-javascript"><code class="lang-javascript">onBeforeUnmount(() => {
  console.log("onBeforeUnmount....");   
});
</code></pre></td></tr><tr><td>onUnmounted</td><td><p>컴포넌트가 해제되고 나서 호출되므로 모든 기능 사용 할 수 없다.<br><br><mark style="color:orange;">* 서버 사이드 렌더링 중에 호출 되지 않음.</mark></p><pre class="language-javascript"><code class="lang-javascript">onUnmounted(() => {
  console.log("onUnmounted....");   
});
</code></pre></td></tr><tr><td>onActivated</td><td>keepAlive태그는 컴포넌트가 다시 렌더링되는 것 막기 위해서 사용되는 것으로 v-is 디렉티브와 같이 사용되며 이것을 사용해서 컴포넌트의 상태가 보존되기 시작하면 호출된다.<br>&#x3C;KeepAlive><br>  &#x3C;componet :is:"activeComponent" /><br>&#x3C;/KeepAlive></td></tr><tr><td>onDeactivated</td><td>KeepAlive 상태가 유지되다 효력이 상실되면 호출된다.</td></tr><tr><td>onRenderTracked</td><td>가상 DOM이 변경 될 때 마다 관찰용으로 호출 ( 개발 모드 전용이며, 서버 사이드 렌더링 중에 호출되지 않음 )<br>renderTracked(e) {<br>  console.log(e.log);<br>}</td></tr><tr><td>onRenderTriggered</td><td>가상 DOM이 반영 될 때 마다 관찰용으로 호출 ( 개발 모드 전용이며, 서버 사이드 렌더링 중에 호출되지 않음 )</td></tr><tr><td>onErrorCaptured</td><td>자식 컴포넌트에서 전파된 에러가 캡쳐되었을 때 호출</td></tr></tbody></table>

```html
<template>
  <h1>{{ propMsg }}</h1>

  <div class="card">
    <button type="button" @click="count++">count is {{ count }}</button>   
    <p>
      Edit
      <code>components/MyComponent.vue</code> 
    </p>
  </div>
</template>

<script setup>
  import { ref, defineProps } from 'vue'             

  import { onBeforeMount, onMounted  } from 'vue'
  import { onBeforeUpdate, onUpdated } from 'vue'
  import { onBeforeUnmount,  onUnmounted } from 'vue'

  const props = defineProps({                        
    msg: String,
  })

  const propMsg = ref(props.msg) ;                    
  const count = ref(0) ;
  

  onBeforeMount(() => {
    console.log(`Composition API : onBeforeMount ::
    count.value = ${count.value}`);   
  });

  
  onMounted(() => {
    console.log(`Composition API : onMounted ::  
    count.value = ${count.value}`); 
  });
  
  onBeforeMount(() => {
    console.log(`Composition API : onBeforeMount ::
    count.value = ${count.value}`);   
  });

  // 생명 주기 훅
  onMounted(() => {
    console.log(`Composition API : onMounted ::  
    count.value = ${count.value}`); 
  });

  onBeforeUpdate(() => {    
    console.log(`Composition API : onBeforeUpdate ::
    oldCount.value = ${count.value}`);  
  });

  onUpdated(() => {
    console.log(`Composition API : onUpdated ::
    count.value = ${count.value}`);  
  });

  onBeforeUnmount(() => {
    console.log(`Composition API : onBeforeUnmount
    count.value = ${count.value}`); 
  });

  onUnmounted(() => {
    console.log(`Composition API : onUnmounted ::
    count.value = ${count.value}`);   
  }); 

</script>

<style scoped> 
</style>
```
