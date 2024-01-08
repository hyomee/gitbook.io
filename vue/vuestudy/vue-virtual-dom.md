# Vue에서  Virtual DOM

## 1. DOM

* DOM(Document Object Model)은 웹 문서용 프로그래밍 인터페이스로 문서의 구조와 내용을 설명하고 문서에 액세스하고 조작하는 방법을 제공하는 문서 개체 모델로 입니다. &#x20;
* DOM은HTM문서를 로드 할 때 HTML 코드 구문을 분석하여  DOM 트리를 생성하고 트리의 각 요소는 문서의 구조와 내용을 변경하기 위해 조작할 수 있는 개체입니다.
* HTML Elements, Attributes, CSS styles, Events, Methods등을제어할 수 있는 표준 인터페이스로 자바스크립트와 HTML 문서를 이어주는 역활을 합니다.

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption><p>HTML DOM  Tree</p></figcaption></figure>

HTML 문서 DOM Tree 예시

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

브라우저의 동작원리를 알고 싶으면 [여기을 읽어](https://d2.naver.com/helloworld/59361) 보세요

## 2. Virtual DOM

* Virtual DOM은 React, Vue.js 및 Elm과 같은 선언적 웹 프레임 워크에서 사용되는 DOM의 경량 JavaScript 표현 으로 "실제" DOM과 동기화되는 프로그래밍 개념입니다.
* DOM 조작이 느리고 비효율적이기 때문에 빠른 Virtual DOM을 사용합니다.&#x20;
* Virtual DOM은 DOM 내부에서 무언가가 변화할 때를 감지 하고, DOM API가 사용 되는 횟수를 제한 하면서, 진짜 DOM에서 업데이트 되어야 할 것이 무엇인지 정확하게 찾습니다.&#x20;
* <mark style="color:green;">**Real DOM과  차이점은 다음과 같습니다.**</mark>
  * <mark style="color:green;">**가상 DOM은 조판되거나 다시 그려지지 않습니다.**</mark>
  * <mark style="color:green;">**가상 DOM은 한 번만 렌더링됩니다.**</mark>
  * <mark style="color:green;">**가상 DOM은 로컬 렌더링을 수정할 수 있습니다.**</mark>
  * <mark style="color:green;">**DOM을 조작하려면 브라우저 호환성 및 기타 문제를 고려해야 하지만 가상 DOM은 다양한 플랫폼에서 사용할 수 있습니다.**</mark>

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

## 3. Vue 에서 Virtual DOM

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption><p>Data 변경 부터 렌더링 까지 라이프 사이클</p></figcaption></figure>

1. Vue에서 name, address의 값을 변경하면 setter 메서드에서 의해서 Watcher에 통지 합니다.
   * Watcher은 컴포넌트 A가 초기화 될 때 생성됩니다.
     * Watcher 열활 : 가상 DOM과 실제 DOM을 갱신하는 역활을 합니다.
   * vue는 defineProperty를 사용하여 컴포넌트 내 data 요소에 대한 setter, getter을 생성  합니다.
   *   Object.defineProperty() 정적 메서드을 사용한 코드의 예시 입니다.\
       \
       참고 : [mdn web docs : Object.defineProperty()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global\_Objects/Object/defineProperty)\


       {% code title="// 브라우저 콘솔에서 확인 할 수 있습니다." %}
       ```javascript
       > var obj = {}; 
       > Object.defineProperty(obj, "text", {
          get: function() {
            return text + "get";
          },
          set: function (newText) {
            text = newText + "set";
          }
         });
       {}

       > obj.text = "text";
       'text'
       > obj.text
       'textsetget'
       ```
       {% endcode %}
2. 통보를 받으면 Watcher는 Queue에 추가 합니다.
   * Queue 추가  이유 :  동일한 Watcher를 여러번 실행하지 않기 위해서
     * name과 address가 동시에 업데이트 되면 두번 update되는 것을 방지 하기 위해  큐에 넣고 사용될 때 특정 순서로 정렬 됩니다.
3. nextTick API를 실행 합니다.
   * 대기열 내의 모든 Watcher를 소비하고 플러시하는 Vue 함수로 모든 Watcher가 소비되고 플러시되면 DOM은 Watcher의 run() 함수에서 업데이트 됩니다.
4. Virtual DOM과 Real  DOM을 함께 랜더링 합니다.
   * Watcher 객체 내부에 저장된 run() 함수 내에서 발생합니다.





참고 : [https://www.template.net/business/document-object-model/](https://www.template.net/business/document-object-model/)
