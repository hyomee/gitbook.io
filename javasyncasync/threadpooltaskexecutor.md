# ThreadPoolTaskExecutor

자바에서 동시성 프로그램을 만들기 위해서는 Runnable객체를 만든 다음 Thread 객체륽 만들어 실행을 해야 하는데 이것은 Thread를 만드는데 비용이 많이들고 새 Thread Instance를 만들기 때문에 성능이 저하된다. 이런 이유로 Thread Pool을 만들어 Thread Instance에 대한 문제를 해결하고자 한다,

## 1. Thread Pool 이란

<figure><img src="../.gitbook/assets/image (216).png" alt=""><figcaption></figcaption></figure>

동시에 여러 작업을 효율적으로 실행 및 관리하기 위해 서버에서 만드는 스레드의 모음으로 새로운 Thread를 만들지 않고 미리 생성된 Thread를 Thread Pool에서 가지고와 재사용하는것으로 성능과 리소스 관리에 도움이 된다.&#x20;

Thread Pool은 병렬처리 되므로 Thread 개수가 증가하여&#x20;
