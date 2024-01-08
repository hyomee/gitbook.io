# 5. 비동기처리

## 1. 동기 , 비동기 이해

동기 처리는 아래 그림 처럼 fnA, fnB, fnC 처럼 순차적으로 실행 되는 것을 의미 합니다. 즉 fnB가 서버를 호출하여 응답을 받은 후에 fnC를 호출 합니다.

<figure><img src="../../.gitbook/assets/image (129).png" alt=""><figcaption><p>동기 </p></figcaption></figure>

비동기 처리는 호출 순서는 아래 그림 처럼 fnA, fnB, fnC이지만 fnB 호출 후 응답을 받기 전에 바로 fnC를 호출 하는 것 입니다. 즉 fnB의 응답이 fnC 처리 후 올수 있습니다.

<figure><img src="../../.gitbook/assets/image (130).png" alt=""><figcaption></figcaption></figure>

#### 동기 와 비동기 차이점

| 구분   | 동기처리                              | 비동기처리                           |
| ---- | --------------------------------- | ------------------------------- |
| 실행순서 | 위에서 아래로 작성된 순서로 응답을 받은 후 다음 명령 실행 | 위에서 아래로 작성된 순서로 실행 되지만 독립적으로 실행 |
| 실행대기 | 응답이 올때 까지 기다림                     | 응답과 관계 없이 바로 실행                 |

## 2. 콜백 패턴

동기식 프로그램에서 일련의 연속적 연산 단계로 코드를 실행 하는 블로킹인데 비동기 프로그래밍에서는 이전 작업 실행이 완료 되지 않아도 다음 작업이 실행이 되는데 이전 작업이 완료 되었을 경우 다음 작업이 실행 할 수 있도록 하는 것이 콜백 패턴 이다.

### 2-1. 연속전달 방식 ( CPS : continuation-processing style )

* 작업이 종료 되었을 때 완료 이후 실행 할 수 있는 함수를 인자로 전달 하는 방식으로 일반적으로 마지막 파라메터에 정의&#x20;
* 단순히 결과를 호출자에게 직접 반환하는 대신 결과를 다른 함수(콜백)로 전달 하는 것을 의미함.&#x20;
* 동기 함수는 조작을 완료 할 때까지 블로킹, 비동기 함수는 제어를 즉시 반환하고 결과는 Event Loop의 다음 사이클에서 핸들러로 전달

#### 2-1-1. 동기식 전달

```javascript
// cps 
function addCps(a, b, callback) {
    callback( a + b );
}

// cps call 함수 
function addCpsPrint(result) {
    console.log("addCpsPrint :: ", result);
}

console.log("이전");
addCps(1,2, result=> console.log( `Result: ${result}`));
addCps(1,2, addCpsPrint);
console.log("이후");
```

<figure><img src="../../.gitbook/assets/image (165).png" alt=""><figcaption><p>결과</p></figcaption></figure>

#### 2-2-2. 비동기식 전달

setTimeout을 사용해서 비동기식 전달 코드드

{% code lineNumbers="true" %}
```javascript
function addCps(a, b, callback) {
    setTimeout( ()=>callback( a + b ) ,1000); // 5)
}

function addCpsPrint(result) {
    setTimeout( 
        function() { // 6)
           console.log("addCpsPrint :: ", result);
        }
    ,1000);
   
}
console.log("이전"); // 1) 
addCps(1,2, result=> console.log( `Result: ${result}`)); // 2)
addCps(1,2, addCpsPrint); // 3)
console.log("이후"); // 4)
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (169).png" alt="" width="500"><figcaption><p>실행 순서</p></figcaption></figure>

* 13 line : 이전 표시
* 14 line : appCps 함수가 실행 되면 Queue 대기열에 넣고 1초 후에 실행
* 15 line : appCps 함수가 실행 되면 Queue 대기열에 넣고 1초 후에 실행
* 16 line : 이후 표시
* 14 line 에 넣었던것 꺼내서 EventLoop 에서 실행
* 15 line 에 넣었던것 꺼내서 EventLoop 에서 실행

<figure><img src="../../.gitbook/assets/image (166).png" alt=""><figcaption><p>결과</p></figcaption></figure>

### 2-2. 비 연속 전달(Non-CPS )

```javascript
const result = [1,3,7].map(e => e - 1);
console.log(result); // [ 0, 2, 6 ]
```
