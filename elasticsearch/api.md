# 검색 API

## **1. ElasticSearch Analyzer & Mapping**&#x20;

### 1-1. analyzer

ElasticSearch( 이하 es ) 의 기본 분석기는 Standard analzer로  공백을 기준으로 문자열을 n개의 토근으로 분리합니다. 추가 해서 필터를 설정 하면 영어인 경우 소문자/대문자로 치환이 가능 합니다. 필터를 하지 않은 Standard analzer를 본다면 ..

```
curl --location --request GET 'https://172.19.164.132:9200/_analyze?pretty=true' \
--header 'Content-Type: application/json' \ 
--data '{
    "analyzer": "standard",
    "text": "elasticsearch spring-data  한글 nori"
}'
결과 :

=================================================================================
{
    "tokens": [
        {
            "token": "elasticsearch",
            "start_offset": 0,
            "end_offset": 13,
            "type": "<ALPHANUM>",
            "position": 0
        },
        {
            "token": "spring",
            "start_offset": 14,
            "end_offset": 20,
            "type": "<ALPHANUM>",
            "position": 1
        },
        {
            "token": "data",
            "start_offset": 21,
            "end_offset": 25,
            "type": "<ALPHANUM>",
            "position": 2
        },
        {
            "token": "한글",
            "start_offset": 27,
            "end_offset": 29,
            "type": "<HANGUL>",
            "position": 3
        },
        {
            "token": "nori",
            "start_offset": 30,
            "end_offset": 34,
            "type": "<ALPHANUM>",
            "position": 4
        }
    ]
}
```

공백을 기준으로 5단어로 분리된 것을 확인 할 수 있습니다. 만약 한글이면 공백을 기준으로 분리되어 단어 기분으로는 맞지 않습니다. 다음 예제는 es에서 기본으로 제공하는 것에 대한 예제 입니다.

```
curl --location --request GET 'https://172.19.164.132:9200/_analyze?pretty=true' \
--header 'Content-Type: application/json' \
--data '{
    "analyzer": "standard",
    "text": "여기는 대한민국입니다."
}'
결과
{
    "tokens": [
        {
            "token": "여기는",
            "start_offset": 0,
            "end_offset": 3,
            "type": "<HANGUL>",
            "position": 0
        },
        {
            "token": "대한민국입니다",
            "start_offset": 4,
            "end_offset": 11,
            "type": "<HANGUL>",
            "position": 1
        }
    ]
}
```

&#x20;한글을 단어로 분리 하기 위해서는 여러가지 형태소 분석기가 있지만 es에서 지원하는 nori 룰 사용 하면 단어로 분리 할 수 있습니다.

```
curl --location --request GET 'https://172.19.164.132:9200/_analyze?pretty=true' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic ZWxhc3RpYzoxbj1lQmFzTTJqR3gxRVpGMDdEWQ==' \
--data '{
    "analyzer": "nori",
    "text": "여기는 대한민국입니다."
}'
결과
{
    "tokens": [
        {
            "token": "여기",
            "start_offset": 0,
            "end_offset": 2,
            "type": "word",
            "position": 0
        },
        {
            "token": "대한",
            "start_offset": 4,
            "end_offset": 6,
            "type": "word",
            "position": 2
        },
        {
            "token": "민국",
            "start_offset": 6,
            "end_offset": 8,
            "type": "word",
            "position": 3
        },
        {
            "token": "이",
            "start_offset": 8,
            "end_offset": 11,
            "type": "word",
            "position": 4
        }
    ]
}
```

자세한 학습은 다음 사이트를 참고 해 주새요 [한국어(nori) 분석 플러그인 | Elasticsearch 플러그인 및 통합 \[7.17\]](https://www.elastic.co/guide/en/elasticsearch/plugins/7.17/analysis-nori.html)

### [ ](https://www.elastic.co/guide/en/elasticsearch/plugins/7.17/analysis-nori.html)1-2. analyzer 과 매핑

es에서 분석한 데이터를 저장(inverted index)되고 검색 할 떄 저장된 값을 바탕으로 검색을 제공 합니다. 또한 필드를 정의 할 떄 text 타입과 keyword 타입은 신중히 결정을 해야 합니다. text는 단어를 문장에서 찾지만 keyword는 keyword analyzer을 사용하여 문장 전체가 같아야 합니다.&#x20;

## **2. Search API**

### 2-1. 전체 요청

http:// 도메인 / index / \_search

```
curl --location 'https://localhost(or ip or domain):9200/tourlist/_search?pretty=true' \
--header 'Content-Type: application/json' \
```

### 2-2 전체 인텍스 검색

http:// 도메인 / index / \_search ? q =  검색어 & pretty

인텍스 전체에서 부산이 포함 되어 있는 모든 필드를 조회 합니다.

```
curl --location 'https://172.19.164.132:9200/tourlist/_search?q=부산pretty' \
--header 'Content-Type: application/json' \
```

### 2-3. 인텍스에서 특정 필드 검색

http:// 도메인 / index / \_search ? q = 컬럼명: 검색어 & pretty

주소 필드에 '부산"이 포함 되어 있는 것 검색

```
curl --location 'https://localhost:9200/tourlist/_search?q=addr1:부산&pretty' \
--header 'Content-Type: application/json' \
```

참고 :  [4.4 검색 API - \_search API - Elastic 가이드북 (kimjmin.net)](https://esbook.kimjmin.net/04-data/4.4-\_search)

참고 : [https://www.elastic.co/guide/en/elasticsearch/reference/7.17/search-your-data.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/search-your-data.html)



&#x20;
