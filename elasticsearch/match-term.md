# match, term

ElasticSearch의 검색 방법 중 하나로 Query Domain Spec Language의 약어로 Query Context와 Filter Context로 분류됩니다.

* Query context :  Full Text Search로 검색어로 문서와 얼마나 매칭되는지 표현하는 score 값을 가집니다\

  * match : 인텍스 매핑 시 text Type로 매핑 ( 분석기 사용 )
  * term : 인텍스 매핑시 field를 keyword, number Type 등으로 매핑 ( 정확도를 위해 권장)
* Filter context : 검색어가 문서에 존재하는지 여부의 형태로 Yes or No로 응답합니다.

***

* 1\. Query context  - Full Text Query - 분석기에 의한 토큰화&#x20;
  * 1-1 match : 전체 텍스트 중에서 특정 용어나 용어들을 검색, 두 단어를 사용 하여 복합 검색 가능 ( or )
  * 1-2 match-phrase : 2개 이상의 단어가 순서까지 비슷하게 검색, 리소스 많이 소요됨&#x20;
  * 1-3 muti-match : 다수의 필드를 match 수행&#x20;
  * 1-4 query\_string : and / or 사용 \[ 예: (연대) AND (벽화) ]
* 2\. Query context  - Term Level Query - 분석기에 의해 토큰화되지 않음 - 일치 &#x20;
  * 2-1 term/terms : 검색어와 일치한 단어가 있는 지 검색
  * 2-2. range : 날짜/숫자/IP 타입의 데이터는 범위 쿼리가 가능, 문자형, 키워드 타입의 데이터에는 범위 쿼리를 사용할 수 없음
  * 2-3. Wildcard : 하나 또는 하나이상의 문자와 매핑 되는 것을 검색

## **1. Query context  - Full Text Query**&#x20;

전문 쿼리에 속하기 때문에 검색어인 분석기에 의한 토큰화가 되어 매칭이 됩니다.

| 종류            | 내용                                                                         |
| ------------- | -------------------------------------------------------------------------- |
| match         | 검색어를 문자열 analyzer를 통해 분석한 토근을 가지고 있는 문서 검색                                 |
| match-phrase  | match와 비슷하지만 검색어가 순서와 동일한 것을 검색                                            |
| muti-match    | 다수의 필드로 검색 ( 검색에 대한 스코어 계산은 필드에 대해서 개별 스코어를 구한 다음에 그중 가장 큰 값을 대표 스코어로 구함 ) |
| query\_string | and 와 or 같이 검색어 간 연산이 필요할 떄                                                |

### 1-1 match&#x20;

tourlist 인텍스에 있는 title 필터의 값이 anlyzer를 통해서 분석 한 토근이 일치하는 문서를 score가 가장 높은 문서를 최상단에 올려 표시합니다.&#x20;

```
curl --location --request GET 'https://localhost:9200/tourlist/_search?pretty=null' \
--header 'Content-Type: application/json' \ 
--data '{
    "query": {
        "match": {
            "title": "연대 벽화"
        }
    }        
}'

결과
{
    "took": 4,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 1.8299086,
        "hits": [
            {
                "_index": "tourlist",
                "_id": "5nEMRIoBX0WMIirXSEkB",
                "_score": 1.8299086,
                "_source": {
                    ...
                    "title": "가덕도 연대봉",
					...
                }
            },
            {
                "_index": "tourlist",
                "_id": "4XEMRIoBX0WMIirXRUlW",
                "_score": 1.633847,
                "_source": {
                    "_class": "com.hyomee.service.elastic.tour.doc.TourListDoc",
                    ...
                    "title": "가고파 꼬부랑길 벽화마을", 
					...
                }
            }
        ]
    }
}
```

### 1-2 match-phrase

tourlist 인텍스에 있는 title 필터의 값이 입력된 검색어의 순서와 동일한 토근을 문장에서 찾아서 도려 줍니다,

```
curl --location --request GET 'https://192.168.210.52:9200/tourlist/_search?pretty=null' \
--header 'Content-Type: application/json' \
--data '{
    "query": {
        "match_phrase": {
            "title": "벽화 꼬부랑"
        }
    }        
}'
```

검색이 되지 않습니다. 검색되지 않는 이유를 알아보기 위해 문장 분석을 하면 다음과 문장 분석이 "갈(가고파), "꼬부랑", "길", "벽화", "마을"로 inverted index 토근이 생성됩니다.

```
curl --location --request GET 'https://192.168.210.52:9200/_analyze?pretty=null' \
--header 'Content-Type: application/json' \
--data '{
    "analyzer": "nori",
    "text": "가고파 꼬부랑길 벽화마을"
}'

결과
{
    "tokens": [
        {
            "token": "갈",
            "start_offset": 0,
            "end_offset": 1,
            "type": "word",
            "position": 0
        },
        {
            "token": "꼬부랑",
            "start_offset": 4,
            "end_offset": 7,
            "type": "word",
            "position": 2
        },
        {
            "token": "길",
            "start_offset": 7,
            "end_offset": 8,
            "type": "word",
            "position": 3
        },
        {
            "token": "벽화",
            "start_offset": 9,
            "end_offset": 11,
            "type": "word",
            "position": 4
        },
        {
            "token": "마을",
            "start_offset": 11,
            "end_offset": 13,
            "type": "word",
            "position": 5
        }
    ]
}
```

따라서 match\_phrase는 순서에 맞아야 하므로 검색이 되지 않습니다. 다름과 같이 변경을 하면 검색이 됩니다, 즉 match\_phrase는 검색어 순서 대로 맞아야 되는 것입니다.

```
curl --location --request GET 'https://192.168.210.52:9200/tourlist/_search?pretty=null' \
--header 'Content-Type: application/json' \
--data '{
    "query": {
        "match_phrase": {
            "title": "꼬부랑 벽화"
        }
    }        
}'
```

### 1-3 muti-match

두 개 이상의 필드에서 match 쿼리를 수행하여 결과를 표시합니다. 아래  tile, overview 필드에 "경기", "벽화" 단어가 포함된 inverted index 토근을 찾아서 결과를 표시하는 예제입니다.

```
curl --location 'https://192.168.210.52:9200/tourlist/_search?pretty=null' \
--header 'Content-Type: application/json' \
--data '{
    "query": {
        "multi_match": {
            "query": "경기 벽화", 
            "fields": ["title", "overview"]
        }
    }        
}'
```

### 1-4 query\_string

and/or , 와일드카드(\*)를 사용해서 검색을 합니다.

title 필드에 "연대", "벽화"가 포함된 문서 검색 예제입니다.

```
curl --location --request GET 'https://192.168.210.52:9200/tourlist/_search?pretty=true' \
--header 'Content-Type: application/json' \ 
--data '{
    "query": {
        "query_string": {
            "fields": ["title"],
            "query": "연대 벽화"
        }
    }        
}'
```

title 필드에 "연대" AND/OR "벽화"가 포함된 문서 검색 예제입니다.

```
curl --location --request GET 'https://localhost:9200/tourlist/_search?pretty=null' \
--header 'Content-Type: application/json' \ 
--data '{
    "query": {
        "query_string": {
            "fields": ["title"],
            "query": "(연대) AND (벽화)"
        }
    }        
}'
```

title 필드에 와일드카드 검색

```
curl --location --request GET 'https://localhost:9200/tourlist/_search?pretty=null' \
--header 'Content-Type: application/json' \ 
--data '{
    "query": {
        "query_string": {
            "fields": ["title"],
            "query": "*부랑*"
        }
    }        
}'
```

&#x20;

## **2. Query context  - Term Level Query**&#x20;

분석기에 의해 토큰화되지 않는다. 즉 분석기를 거치지 않았기 때문에 대소문자도 정확히 맞아야 합니다.

| 종류       | 설명                              |
| -------- | ------------------------------- |
| term     | 검색어와 일치한 단어가 있는 지 검색            |
| terms    | 여러개의 검색어와 일치 한 단어가 있는 지 검색      |
| range    | <p>특정 범위 안에 있는 값 검색<br><br></p> |
| wildcard | 와일드 카드 패턴에 해당 되는 값 검색           |

### 2-1 term/terms

```
// title : 가덕도 연대봉

curl --location --request GET 'https://192.168.210.52:9200/tourlist/_search?pretty=null' \
--header 'Content-Type: application/json' \ 
--data '{
    "query": {
        "term": {
            "title": "연대봉"
        }
    }        
}'
```

"가덕도 연대봉" 에서 :"연대봉"을 찾는 것인데 위 예제를 수행하면 결과는 없습니다. noti\_analyzer에서 inverted index로 토근에 "연대", "봉"으로 되어 있어서 같은 단어가 없어서입니다. 한글 사용 시 주의가 필요합니다. 아래와 같이 수정하면 검색이 됩니다.

```
curl --location --request GET 'https://localhost:9200/tourlist/_search?pretty=null' \
--header 'Content-Type: application/json' \ 
--data '{
    "query": {
        "term": {
            "title": "연대"
        }
    }        
}'
```

term는 여러개의 검색어로 검색합니다,

```
curl --location 'https://localhost:9200/tourlist/_search?pretty=null' \
--header 'Content-Type: application/json' \ 
--data '{
    "query": {
        "terms": {
            "overview": ["부산" ,"벽화"]
        }
         
    }        
}'
```

참고 : [https://www.elastic.co/guide/en/elasticsearch/reference/7.17/query-dsl-term-query.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/query-dsl-term-query.html)

참고 : [https://www.elastic.co/guide/en/elasticsearch/reference/7.17/query-dsl-terms-query.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/query-dsl-terms-query.html)

### 2-2. range

특정 범위에 대한 검색으로 날짜/숫자/IP 타입의 데이터는 범위 쿼리가 가능, 문자형, 키워드 타입의 데이터에는 범위 쿼리를 사용할 수 없으며 다음과 같은 매개 변수가 있습니다

* gt(선택 사항) 보다 큼.
* gte(선택 사항) 크거나 같습니다.
* lt(선택 사항) 미만.
* lte(선택 사항) 작거나 같습니다.&#x20;

자세한 항목은 참고 : [https://www.elastic.co/guide/en/elasticsearch/reference/7.17/query-dsl-range-query.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/query-dsl-range-query.html)

```
curl --location --request GET 'https://192.168.210.52:9200/tourlist/_search?pretty=null' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic ZWxhc3RpYzpHbEZBSk8yST0qKmtrOGhqUDAyZw==' \
--data '{
    "from": 0,
    "size": 3,
    "query":  {
        "range": {
            "createdtime" : {
                "gte" : "20211129222823"
            }
        }
    }        
}'
```

### 2-3. Wildcard

하나 또는 하나이상의 문자와 매핑되는 것을 검색합니다.

참고 : [https://www.elastic.co/guide/en/elasticsearch/reference/7.17/query-dsl-wildcard-query.html#query-dsl-wildcard-query](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/query-dsl-wildcard-query.html#query-dsl-wildcard-query)

예제) title 필드에 가덕이 포함된 인텍스 검색

```
curl --location --request GET 'https://localhost:9200/tourlist/_search?pretty=null' \
--header 'Content-Type: application/json' \ 
--data '{
    "query":  {
        "wildcard": {
            "title" : "*가덕*"
        }
    }        
}'
```

&#x20;

Reqexp, Exists 등 이 있으며 이것을 term-level queries라 하며 자세한 학습은 다음 링크에서 확인할 수 있습니다

참고 : [https://www.elastic.co/guide/en/elasticsearch/reference/7.17/term-level-queries.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/term-level-queries.html)

참고 : [https://www.elastic.co/guide/en/elasticsearch/reference/7.17/query-dsl.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/query-dsl.html)



&#x20;
