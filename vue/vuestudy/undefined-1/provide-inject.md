# Provide / Inject

부모 컴포넌트에서 자식 컴포넌트로 데이터를 전달해야 할 때 props를 사용하여 데이터를 전달하는데 컴포넌트의 깊이가 깊은 경우 props로 넘기는 것에는 한계가 있는데 이 점을 극복하고자 만든 것이 Provide / Inject 입니다.

* provide : 컴포넌트의 하위 항목에 데이터를 제공하려면 사용해야 하는 함수
* Inject : 부모 컴포넌트에서 제공하는 데이터를 주입하려면 사용해야 하는 함수

| props                                                                                                                                                                                                                                                         | provide/inject                                                                                                                                                                                                                                                                               |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <img src="../../../.gitbook/assets/image (234).png" alt="" data-size="original">                                                                                                                                                                              | <img src="../../../.gitbook/assets/image (235).png" alt="" data-size="original">                                                                                                                                                                                                             |
| <pre class="language-javascript"><code class="lang-javascript">// 파라메터 값 전달
&#x3C;MyComponent :message="안녕!" />

&#x3C;script setup>
// 전달된 값 받음
const props = 
        defineProps(['message']);

console.log(props.message)
&#x3C;/script>



</code></pre> | <pre class="language-javascript"><code class="lang-javascript">// 값 생성
&#x3C;script setup>
import { provide } from 'vue'

provide('message', '안녕!')
&#x3C;/script>

// 값 받음
&#x3C;script setup>
import { inject } from 'vue'

const message = inject('message')
&#x3C;/script>
</code></pre> |

