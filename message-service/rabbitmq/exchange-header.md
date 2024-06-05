# Exchange-Header

**메시지 헤더를 통해 binding key를 사용하는 것보다 더 다양한 속성을 사용할 수 있으며 헤더 값이 바인딩 시 지정된 값과 같은 경우에만 일치하는 것으로 간주합니다.**

* 헤더 교환은 **일치하는 메시지 헤더**에 따라 메시지를 라우팅하며 라우팅 키는 무시합니다.
* 헤더와 같은 JSON을 사용하여 헤더 교환(my-header-exchange)에 바인딩됨

**Topic  Exchange 흐름**

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

* 허용 되는 해더(x-match)에는 any, all 두가지 유형이 있습니다.
  * any: Exchange로 보내는 메시지에 Queue가 연결된 헤더 중 하나 이상이 포함되어야 함을 의미
  * all: Exchange로 보내는 메시지에 Queue가 연결된 모든헤더 가 같아야 함을 의미

## 1. 관리자 UI

## 2.&#x20;
