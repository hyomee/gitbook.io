# ElasticSearch Index settings 변경

## 1. 문제점

Spring Data Elasticsearch를 이용해서 문서 생성시 analyzer를 별도로 설정 하지 않고 analyzer = "nori"를 설정하여 기본값을 사용하면 단어 검색시 "국제터미널"을 "국제", "터미널"로 분리 되는데 이것을  "국제터미널"로 하고자 한다.

## 2. 해결방안

형태소 분석 노리는  명사 기준으로 분리하여서 발생하는 문제로 custom 분석기를 만들기 전에 nori\_tokenizer에 대해서 [nori tokenizer 문서](https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-nori-tokenizer.html) 통해서  확인 할 수 있다.   그중 decompound\_mode 는  분해 모드는 토크나이저가 복합 토큰을 처리하는 방법을 결정하는 요소로 각 의미는 다음과 같다.

| Setting | 의미                         | 예제                                                                      |
| ------- | -------------------------- | ----------------------------------------------------------------------- |
| none    | 복합 명사로 분리하지 않습니다.          | <p></p><ul><li>가거도항 => 가거도항</li><li>가곡역 => 가곡역</li></ul>                |
| discard | 복합 명사로 분리하고 원본 데이타는 삭제합니다. | <p></p><ul><li>가거도항 => 가거도, 항</li><li>가곡역 => 가곡, 역</li></ul>            |
| mixed   | 복합 명사로 분리하고 원본 데이타도 남겨둡니다. | <p></p><ul><li>가거도항 => 가거도항, 가거도, 항</li><li>가곡역 => 가곡역, 가곡, 역</li></ul> |

### 2-1. 작업 순서

#### 2-1-1.  설정 변경하고자 하는 index를 닫는다.

```
https://domain or Ip:9200/index명/_close
```

예시 :&#x20;

````bash
curl --location --request POST 'https://192.168.210.52:9200/wordsearch/_close' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic ZWxhc3RpYzpHbEZBSk8yST0qKmtrOGhqUDAyZw==' \
--data ''
```
````

#### 2-1-2 설정 변경을 한다.

```
https://domain or Ip:9200/index명/_settings
```

analysis 영역에 대해서 analyzer/tokenizer를 설정 한다.  다음과 같이 작성이 되어 있다면&#x20;

```java
@Document(indexName = "wordsearch") 
public class WordSearchDoc {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private String wordSearchId;

    @Field(type = FieldType.Text, analyzer = "nori", fielddata = true)
    private String words;

}
```

analyzer = "nori" 부분의 이름과 analyzer의 명을 맞추고 "tokenizer"의 명을 사용자 정의 후 "tokenizer"를 사용자 정의한 명으로 하고 decompound\_mode를 설정 하면 된다. 아래 예제는 mixed 모드로 설정을 한 예시이다.&#x20;

```bash
curl --location --request PUT 'https://192.168.210.52:9200/wordsearch/_settings' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic ZWxhc3RpYzpHbEZBSk8yST0qKmtrOGhqUDAyZw==' \
--data '{
  "settings": {
    "analysis": {
      "analyzer": {
        "nori": {
          "tokenizer": "tl_tokenizer"
        }
      },
      "tokenizer": {
        "tl_tokenizer": {
          "type": "nori_tokenizer",
          "decompound_mode": "mixed"
        }
      }
    }
  }
}'
 
```

#### 2-1-3. 닫힌 index를 오픈 한다.

```
https://domain or Ip:9200/index명/_open
```

예시:

```bash
curl --location --request POST 'https://192.168.210.52:9200/wordsearch/_open' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic ZWxhc3RpYzpHbEZBSk8yST0qKmtrOGhqUDAyZw==' \
 
```

## 3. 향후 과제

@Setting(settingPath = "설정.json")을 사용 해서 Document 생성 시점에 설정 하도록 코드를 변경해 보자
