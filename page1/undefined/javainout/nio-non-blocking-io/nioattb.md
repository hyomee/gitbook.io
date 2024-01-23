# 기본개념

## 3. Selector

스레드는 운영 채제애 비용을 먾이 들이는 작업으로 하나의 스레드에서 여러 채널을 관리하며 비용 소모가 적게 들게 되니다. 이런 경우 Selector에 여러개의 스레드를 묶어서 하나의 스레드로 처리하게 됩니다.

<figure><img src="../../../../.gitbook/assets/image.png" alt="" width="306"><figcaption></figcaption></figure>



### 4-1. selector 셍성&#x20;

```java
Selector selector = Selector.open();
```
