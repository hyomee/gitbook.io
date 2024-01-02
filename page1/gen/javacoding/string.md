# String 사용에 대한 이야기

## 1. String은 gc에 영향을 준다. 왜 ?

프로그램 코드 작성시 가장 많이 사용하는 객체가 String, Collection이다. 특히 String 연산은 코드에서 여러가지 이유로 많이 사용 한다.&#x20;

다음 코드를 보자

```java
final String aValue = "abcde";
String a = new String();
for ( int index = 0 ; index < 10000 ; index ++ ) {
    a += aValue;
}
```

**a의 메모리 사용량은 어떻게 될까 ?**

| <p>첫번째 수행 : a += aValue <br><br>723 번지 : gc 대상</p> | <img src="../../../.gitbook/assets/image (13).png" alt="" data-size="original"> |
| -------------------------------------------------- | ------------------------------------------------------------------------------- |
| <p>두번째 수행 : a += aValue<br><br>725 번지 : gc 대상</p>  | <img src="../../../.gitbook/assets/image (15).png" alt="" data-size="original"> |
| <p>세번째 수행 : a += aValue<br><br>.....</p>           | <img src="../../../.gitbook/assets/image (16).png" alt="" data-size="original"> |

데이터 크기도 커지면서 주소 할당을 하고 있다. 사용하지 않는 주소는 gc대상 입니다.

## 2. StringBuffer or StringBuilder로 변경

```java
final String aValue = "abcde";
StringBuffer a = new StringBuffer();
for ( int index = 0 ; index < 10000 ; index ++ ) {
    a.append(aValue);
}
```

| 첫번째 수행 : a.append(aValue)  | <img src="../../../.gitbook/assets/image (17).png" alt="" data-size="original"> |
| -------------------------- | ------------------------------------------------------------------------------- |
| 두번째 수행 : a.append(aValue)  | <img src="../../../.gitbook/assets/image (18).png" alt="" data-size="original"> |
| 세번째 수행 : a.append(aValue)  | <img src="../../../.gitbook/assets/image (19).png" alt="" data-size="original"> |

주소는 변경 되지 않고 데이터 만 변경되어 들어 간다.&#x20;

## 3. StringBuffer와 StringBuilder를 어떻게 사용해야 하나

* StringBuffer
  * 스레드에 안전한 프로그램이 필요할 때 사용&#x20;
  * 즉. 멀티쓰레드환경에서 synchronized키워드가 가능하므로 동기화가 가능
  * static으로 선언한 문자열 변경 또는 singleton으로 선언된 클래스에 선언된 문자열&#x20;
* StringBuilder
  * 스레드에 안전한지 여부와 전혀 관계 없는 코드
  * 즉. 동기화를 지원하지 않기 때문에 멀티 쓰레드 환경에서는 적합하지 않다.
  * 메서드내 지역 변수
