# 옵션 API 생명주기

참고 : [옵션: 생명주기](https://ko.vuejs.org/api/options-lifecycle.html)

<table><thead><tr><th width="174.33333333333331">옵션 API</th><th>설명</th></tr></thead><tbody><tr><td>beforeCreate</td><td>컴포넌트를 생성하기 전이므로 data, watch 같은 함수는 동작 하지 않는다.</td></tr><tr><td>create</td><td>컴포넌트를 생성되면 호출되므로 data를 초기화 할 수 있다</td></tr><tr><td>beforeMount</td><td>render 함수를 호출 하기 전으로 renderTracked으로 관찰 할 수 있다</td></tr><tr><td>mounted</td><td>DOM 엘리먼트로 마운트 된 후 호출되므로 엘리먼트 참조가 가능한다. 즉 ref와 같은 함수를 통해서 작성한 변수를 참조 할 수 있다. onRenderTracked 훅 호출 된다, ( <strong>가상 DOM -> DOM 반영</strong> )</td></tr><tr><td>beforeUpdate</td><td>데이터가 변경되어 DOM에 반영 되기 전 호출됨으로 엘리머크를참조하는수변 참조는 이전 값</td></tr><tr><td>updated</td><td>데이터가 변경되어 DOM에 변경 완료되는 시점에 호출하므로 참조된 변수를 제어 할 수 있다. 단 자식 노드가 있으면 nextTick을 사용해서 자식 업데이트가 완료될 때 까지 기다려야 한다.<br>update() {<br>   this.$nextTick(function() {<br>     // 모든 자식 노드 업데이트 됨<br>    })<br>}</td></tr><tr><td>beforeUnmount</td><td>컴포넌트가 해제 되기 전에 호출되므로 모든 기능 사용 할 수 있다.</td></tr><tr><td>unmounted</td><td>컴포넌트가 해제되고 나서 호출되므로 모든 기능 사용 할 수 없다.</td></tr><tr><td>activated</td><td>keepAlive태그는 컴포넌트가 다시 렌더링되는 것 막기 위해서 사용되는 것으로 v-is 디렉티브와 같이 사용되며 이것을 사용해서 컴포넌트의 상태가 보존되기 시작하면 호출된다.<br>&#x3C;KeepAlive><br>  &#x3C;componet :is:"activeComponent" /><br>&#x3C;/KeepAlive></td></tr><tr><td>deactivated</td><td>KeepAlive 상태가 유지되다 효력이 상실되면 호출된다.</td></tr><tr><td>renderTracked</td><td>가상 DOM이 변경 될 때 마다 관찰용으로 호출 ( 개발 모드 전용이며, 서버 사이드 렌더링 중에 호출되지 않음 )<br>renderTracked(e) {<br>  console.log(e.log);<br>}</td></tr><tr><td>renderTriggered</td><td>가상 DOM이 반영 될 때 마다 관찰용으로 호출 ( 개발 모드 전용이며, 서버 사이드 렌더링 중에 호출되지 않음 )</td></tr><tr><td>errorCaptured</td><td>자식 컴포넌트에서 전파된 에러가 캡쳐되었을 때 호출</td></tr></tbody></table>

```html
<template>
  <h1>{{ propMsg }}</h1>

  <div class="card">
    <button type="button" @click="count++">count is {{ count }}</button>   
    <p>
      Edit
      <code>components/MyComponentOption.vue</code> 
    </p>
  </div>
</template>

<script>
  export default {
    props: {
      msg: {
        type: String,
        default: "자식 vue"
      }
    },
    // data() { ... };
    data: function() {
      return {
        propMsg: this.msg,
        count : 0
      }
    },

    // 생명 주기
    beforeCreate() {       
      console.log(`beforeCreate() : 
        인스턴스가 초기화된 후 호출 :
        this.count = ${this.count}`); 
    },

    created() { 
      console.log(`created() : 
        인스턴스가 모든 상태 관련 옵션 처리를 완료한 후 호출:
        this.count = ${this.count}`); 
    },

    beforeMount() { 
      console.log(`beforeMount() : 
        컴포넌트가 마운트되기 직전 호출:
        this.count = ${this.count}`); 
    },

    mounted() {
      console.log(`mounted() : 
      컴포넌트가 마운트된 후 호출 :
      ${this.count}`);
    },

    beforeUpdate() { 
      console.log(`beforeUpdate() : 
        반응형 상태 변경에 의한 컴포넌트 DOM 트리 업데이트 직전 호출:
        this.count = ${this.count} `);
    },

    updated() { 
      console.log(`updated() : 
        반응형 상태 변경에 의한 컴포넌트 DOM 트리 업데이트 후 호출:
        this.count = ${this.count} `);
    },

    beforeUnmount() { 
      console.log(`beforeUnmount() : 
        컴포넌트 인스턴스가 마운트 해제되기 직전 호출:
        this.count = ${this.count} `);
    },

    unmounted() { 
      console.log(`unmounted() : 
        컴포넌트가 마운트 해제된 후 호출: 
        this.count = ${this.count} `);
    } ,

    beforeDestroy: function() {
      console.log(`beforeDestroy() : 
        뷰 인스턴스가 제거되기 직전에 호출되는 단계:
        this.count = ${this.count}  `);
    },
     
    destroyed: function() {
      console.log(`destroyed() : 
        뷰 인스턴스가 제거되고 나서 호출되는 단계:
        this.count = ${this.count} `);
    }
  }
                              
</script>

<style scoped> 
</style>
```
