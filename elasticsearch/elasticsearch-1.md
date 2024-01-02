# Elasticsearch 문서 색인 / 조회

* 설치 확인
* 1\. index 생성 : PUT index명
* 2\. index 삭제 : DELETE index명
* 3\. schena(mapping) 확인 : GET index명
* 4\. 검색
  * 4-1. 전체 검색 : GET  /index명/\_search?pretty
  * 4-2. 특정 문자가 포함된 검색  : GET  /index명/\_search?q=부산\&prettyty
  * 4-3. 특정 필드에 있는 값 검색 :  GET /index명/\_search?q=title:부산\&pretty  or match
  * 4-4. 특정 필드의 범위 검색 : GET /index명/\_search?pretty  body : match, filter&#x20;

설치 확인을 위해서 브라우저 또는 Postman, curl을 이용해서 설치 확인을 할 수 있습니다.

curl --location 'https://localhost:9200'&#x20;

```
{
    "name": "Hxxxxxxx-PC", // 노드 이름 				
    "cluster_name": "elasticsearch", // 클러스터 이름
    "cluster_uuid": "HFu_CvuTRIuneGSuxiONdQ",	
    "version": {
        "number": "8.7.1", // 버젼
        "build_flavor": "default",
        "build_type": "tar", // 설치 형태
        "build_hash": "f229ed3f893a515d590d0f39b05f68913e2d9b53",
        "build_date": "2023-04-27T04:33:42.127815583Z",
        "build_snapshot": false,
        "lucene_version": "9.5.0",
        "minimum_wire_compatibility_version": "7.17.0",
        "minimum_index_compatibility_version": "7.0.0"
    },
    "tagline": "You Know, for Search"
}
```

&#x20;

## **1. 인텍스 생성**&#x20;

RestAPI 메서드를 사용하여 인텍스를 생성할 수 있습니다. ( PUT : 신규, POST : 수정, DELETE : 삭제, GET 조회 )

```
Method : PUT
Query  : /tourlist  	// index 명 			
Header : Content-Type: application/json; 
Body   : {
	"mappings": {
		"properties": {
			"zipcode": {
				"type": "text"
			},
			...
		}
	},
	"settings": {
		"index": {
			"number_of_shards": "1",
			"number_of_replicas": "1",
			"refresh_interval": "1s"
		}
	}
}

예) curl --location --request PUT 'https://192.168.210.52:9200/tourlist' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic ZWxhc3RpYzpHbEZBSk8yST0qKmtrOGhqUDAyZw==' \
--data '{
    "mappings": {
        "properties": {
            "zipcode": {
                "type": "text"
            },
            ...
        }
    },
    "settings": {
        "index": {
            "number_of_shards": "1",
            "number_of_replicas": "1",
            "refresh_interval": "1s"
        }
    }
}'
```

spring-boot-starter-data-elasticsearch를 사용 해서 인텍스를 생성하는 것은 @Document(indexName = tourlist) 로 선언하면 됩니다,. 다음은 data-elasticsearch를 사용 해서 인텍스 생성을 한 전문 내역 입니다.

```
PUT /tourlist
Parameters: 
Headers: [User-Agent: elastic-java/8.7.1 (Java/17.0.8), 
		  Accept: application/vnd.elasticsearch+json; 
          compatible-with=8, 
          Content-Type: 
          application/vnd.elasticsearch+json; 
          ...]
Request body: {
	"mappings": {
		"properties": {
			"zipcode": {
				"type": "text"
			},
			....
		}
	},
	"settings": {
		"index": {
			"number_of_shards": "1",
			"number_of_replicas": "1",
			"refresh_interval": "1s"
		}
	}
}
```

필드를 추가하면 자동으로 색인되고 스키마도 추가할 수 있습니다

## **2, 인텍스 삭제**

```
Method  : DELETE
RestAPI : https://localhost(or IP):9200/INDEX명

예) DELETE https://localhost:9200/INDEX명
```

&#x20;

## **3. 스키마 확인**&#x20;

```
Method : GET
QUERY  : /인텍스명 

예) curl --location 'https://192.168.210.52:9200/tourlist' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic ZWxhc3RpYzpHbEZBSk8yST0qKmtrOGhqUDAyZw=='

결과 :
{
    "tourlist": {
        "aliases": {},
        "mappings": {
            "properties": {
                "_class": {
                    "type": "keyword",
                    "index": false,
                    "doc_values": false
                },
                "addr1": {
                    "type": "text"
                },
                "contentid": {
                    "type": "text"
                },
                "overview": {
                    "type": "text"
                },
                "title": {
                    "type": "text"
                },
                "tourListId": {
                    "type": "keyword"
                },
                "zipcode": {
                    "type": "text"
                }
            }
        },
        "settings": {
            "index": {
                "routing": {
                    "allocation": {
                        "include": {
                            "_tier_preference": "data_content"
                        }
                    }
                },
                "refresh_interval": "1s",
                "number_of_shards": "1",
                "provided_name": "tourlist",
                "creation_date": "1693290971652",
                "number_of_replicas": "1",
                "uuid": "A6fo4HtsTSaDh0lBbGJ9uA",
                "version": {
                    "created": "8070199"
                }
            }
        }
    }
```

&#x20;

## **4. 검색**

검색 시 마지막애 쿼리 스트림으로 \&pretty를 넣어 주면 검색 결과를 이쁘게 표시합니다.

* Query Context : 질의에 대한 유사도를 계산하여 결과 이므로 연관성에 대한 score 표시
* Filter Context : 일치 여부에 따른 결과를 하므로 결과에 대한 업데이트가 발생하지 않아 캐시를 사용할 수 있습니다. \
  기본적으로 힙 메모리의 10% 사용하고 있으며, 빠른 검색을 위해서는 Filter Context를 사용하면 됩니다.

### 4-1. 전체 검색

아무 조건 없이 검색을 하면 10개의 검색합니다.

```
curl --location 'https://192.168.210.52:9200' \
--header 'Authorization: Basic ZWxhc3RpYzpHbEZBSk8yST0qKmtrOGhqUDAyZw=='
```

```
{
    "took": 7,	// 소요된 시간
    "timed_out": false,
    "_shards": {
        "total": 1,	// 검색에 참여한 사드 갯수
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 11, // 검색 결과 개수
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "tourlist",
                "_id": "y3EbQIoBX0WMIirX90kK",
                "_score": 1.0,
                "_source": {
                    "_class": "com.hyomee.service.elastic.tour.doc.TourListDoc",
                    "contentid": "2019720",
                    "addr1": "경상남도 창원시 마산합포구 성호서7길 15-8",
                    "addr2": "",
                    ....
                }
            },
            .....
        ]
    }
}
```

### 4-2. 특정 문자가 포함된 검색&#x20;

Query : \_search?q=검색단어\&pretty

```
curl --location 'https://192.168.210.52:9200/tourlist/_search?q=Type3&pretty=null' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic ZWxhc3RpYzpHbEZBSk8yST0qKmtrOGhqUDAyZw==' \
--data ''
```

### 4-3. 특정 필드에 있는 값 검색

query /  match에 필드명:찾고자 하는 단어로 body에 보내면 됩니다.

```
curl --location --request GET 'https://192.168.210.52:9200/tourlist/_search?pretty=null' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic ZWxhc3RpYzpHbEZBSk8yST0qKmtrOGhqUDAyZw==' \
--data '{
    "query": {
        "match": {
            "title": "가덕산"
        }
    }        
}'
```

### 4-4. 특정 필드의 범위 검색

filter를 사용해서 특정 필드의 범위를 지정하여 조회합니다. \
mlevel의 값이 5보다 큰 모든 정보 검색의 예제입니다.

```
curl --location --request GET 'https://192.168.210.52:9200/tourlist/_search?pretty=true' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic ZWxhc3RpYzpHbEZBSk8yST0qKmtrOGhqUDAyZw==' \
--data '{
    "query": {
        "match": {
            "match_all": {}
        },
        "filter": {
            "range": {
                "mlevel" : {
                    "gte": 5
                }
            }
        }
    }        
}'
```

&#x20;

BM25은 입력한 쿼리 (문장, 복수의 단어 시퀀스 등)에 대하여 해당 문서와의 연관성을 수치화시키는 알고리즘으로 다음 사이트를 참고하여 학습하면 됩니다.

[Practical BM25 - Part 1: How Shards Affect Relevance Scoring in Elasticsearch | Elastic Blog](https://www.elastic.co/kr/blog/practical-bm25-part-1-how-shards-affect-relevance-scoring-in-elasticsearch)

[실용적인 BM25 - 제2부: BM25 알고리즘과 변수 | Elastic Blog](https://www.elastic.co/kr/blog/practical-bm25-part-2-the-bm25-algorithm-and-its-variables)

[Practical BM25 - Part 3: Considerations for Picking b and k1 in Elasticsearch | Elastic Blog](https://www.elastic.co/kr/blog/practical-bm25-part-3-considerations-for-picking-b-and-k1-in-elasticsearch)
