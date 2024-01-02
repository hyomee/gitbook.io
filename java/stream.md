# Stream

프로그램을 코드를 작성 할 떄 데이터를 그룹화 하여 처리 하는 것이 전체 소스의 많은 부분이다., 자바에서 Collection , Map을 사용 해서 데이터를 그룹화 하고 제공하는 API를 이용해서 이 부분을 작성 한다, Java 1.8이상 부터는 데이터 처리 하는데 임시 구현 코드 대신 질의로 표현 하여 Collection 데이터를 처리 할 수 있는데 이것을 Stream이라 한다.

## **1. Collection Interface 특징**&#x20;

<figure><img src="https://blog.kakaocdn.net/dn/bFawI0/btrEH3Yqgoa/kTSzcsc80wJG8qS9iIPCtk/img.png" alt=""><figcaption><p>Java Collection, Map</p></figcaption></figure>

&#x20;&#x20;

<table data-header-hidden><thead><tr><th width="126"></th><th width="145"></th><th></th></tr></thead><tbody><tr><td>인터페이스 </td><td>구현클래스 </td><td>특징 </td></tr><tr><td> Set</td><td> HashSet<br>TreeSet</td><td> 순서를 유지하지 않는 데이터의 집합으로 데이터의 중복을 허용하지 않는다.</td></tr><tr><td> List</td><td> LinkedList<br>Vector<br>ArrayList</td><td> 순서가 있는 데이터의 집합으로 데이터의 중복을 허용한다.</td></tr><tr><td> Queue</td><td> LinkedList<br>PriorityQueue</td><td> List와 유사</td></tr><tr><td> Map</td><td> Hashtable<br>HashMap<br>TreeMap</td><td> 키(Key), 값(Value)의 쌍으로 이루어진 데이터으 집합으로,<br>순서는 유지되지 않으며 키(Key)의 중복을 허용하지 않으나 값(Value)의 중복은 허용한다.</td></tr></tbody></table>

## &#x20;**2. Stream**

&#x20;

API 참고 : [https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)

### 2-1. 특징

* 선언형 : 더 간결하고 가독성이 좋아진다.
* 조립 : 유연성이 좋아진다,
* 병렬화 : 성능이 좋아진다.
* 소비 : 단 한번만 소비 한다.

<figure><img src="https://blog.kakaocdn.net/dn/by6aLf/btrEEXqSKa3/hkz9k69m0lddskayYuL0tK/img.png" alt=""><figcaption></figcaption></figure>

```
List<Integer> transactionsIds = 
    transactions.stream()
                .filter(t -> t.getType() == Transaction.GROCERY)
                .sorted(comparing(Transaction::getValue).reversed())
                .map(Transaction::getId)
                .collect(toList());
```

### &#x20;2-2.외부 반복과  내부 반복

* 외부 반복(external iteration) : Collection Interface를 사용해서 사용자가 직접 요소를 반복 ( for-each )

```
for( Car car : cars) { // cars를 명시적으로 반복
   result.add(car.getName());  // 이름을 추출 하여 리스트에 추가
}
```

* 내부 반복(internal iteration) : Stream을 사용해서 반복을 내부에서 알아서 처리 하고 결과만 받는다.

```
 List<String> names = cars.stream()
            .map(Car::getName) // map 메서드르를 getName 메서드를 파라메터화 해서 이름 추출 
            .collect(Collectors.toList()); // 파이프라인 실행 ( 반복자 필요 없음
```

### 2-3. 스트림 연산&#x20;

#### 2-3-1. 스트림 이용 과정

* 질의를 수행할 소스
* 스트림 파이프라인을 구성랄 중간 연산 연결
* 스트림 파이프라인을 실행하고 결과를 만드는 최종 연산&#x20;

<figure><img src="https://blog.kakaocdn.net/dn/c1nQU0/btrEKUTvpJr/zE5zqs14sSGp8SuIGRJ2OK/img.png" alt=""><figcaption></figcaption></figure>

* stateless : 상태정보를 저장하지 않는 형태
* statefull : 상태정보를 저장&#x20;



참고 : [https://www.geeksforgeeks.org/lambda-expressions-java-8/](https://www.geeksforgeeks.org/lambda-expressions-java-8/)

참고 : [https://www.geeksforgeeks.org/stream-in-java/](https://www.geeksforgeeks.org/stream-in-java/)

참고 : [https://www.geeksforgeeks.org/java-8-stream-tutorial/](https://www.geeksforgeeks.org/java-8-stream-tutorial/)

참고 : [https://www.baeldung.com/java-collectors-tomap](https://www.baeldung.com/java-collectors-tomap)

참고 : [https://www.baeldung.com/java-stream-immutable-collection](https://www.baeldung.com/java-stream-immutable-collection)
