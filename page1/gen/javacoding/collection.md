# Collection 객체는 언제 ?

## 1. Set, List

<figure><img src="../../../.gitbook/assets/image (22).png" alt="" width="563"><figcaption></figcaption></figure>

<table><thead><tr><th width="131">interface</th><th width="635">구현체</th></tr></thead><tbody><tr><td>set</td><td><p>중복이 없는 집합 객체을 만들때</p><ol><li><strong>HashSet</strong> : 순서 없이 저장</li><li><p><strong>TreeSet</strong> : red-black이라는 트리에 데이터 담음</p><ul><li>값에 따라서 순서가 정해짐</li><li>저장 되면서 동시에 정렬하기 때문에 HashSet 보다 성능 느림</li></ul></li><li><strong>LinkedHashSet</strong> : 저장된 순서에 따라서 순서 결정</li></ol></td></tr><tr><td>list</td><td><ol><li><strong>Vector</strong> : 크기를 지정할 필요가 없는 배열 클래스</li><li><strong>ArrayList</strong> : Vector 와 비슷하지만 동기화 처리가 되지 않음</li><li><strong>LinkedList</strong> : ArrayList 와 비슷하지만 Queue 인터페이스를 구현했기 때문 FIFO 큐 작업 수행</li></ol></td></tr></tbody></table>

&#x20;참고 : [https://www.edureka.co/blog/java-collections/](https://www.edureka.co/blog/java-collections/)

## 2. Map

<figure><img src="../../../.gitbook/assets/image (21).png" alt="" width="371"><figcaption></figcaption></figure>

<table><thead><tr><th width="131">interface</th><th width="635">구현체</th></tr></thead><tbody><tr><td>map</td><td><ol><li>Hashtable : 내부에서 관리하는 해쉬테이블 객체가 동기화 되어 있으므로 동기화 필요한 부분에서 사용</li><li>HashMap : null값 허용되고 동기화 되지 않음</li><li><p>ThreeMap : red-black이라는 트리에 데이터 담음</p><ul><li>ThreeSet과 다른점은 Key에 의해서 순서가 정해짐</li></ul></li><li>LinkedHashMap : HashMap 과 동일 하며 이중 연결 리스트 방식을 사용하여 저장</li></ol></td></tr></tbody></table>

## 3. 성능을 고려한 사용

* Set 계열  : 데이터를 순서에 따라 탐색하는 작업은 TreeSet 사용이 좋으며 그럴 필요가 없을 때는 HashSet **,** LinkedHashSet 을 사용 한다.
* List 계열 : 데이터의 삭제 작업이 있으면 LinkedList를 사용하고 없으면 Vector or ArrayList 사용
* Map 계열 : ThreeMap 이 가장 느림

## 4. Sun에서 권장&#x20;

* Set 계열  : HashSet
* List 계열 : ArrayList
* Map 계열 : HashMap
* Queue 계열 : LinkedList
