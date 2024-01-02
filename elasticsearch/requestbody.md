# RequestBody 검색

**RequestBody 검색**

http:// 도메인 / index / \_search ? pretty

| 옵션        | 내용                                                                      |
| --------- | ----------------------------------------------------------------------- |
| query     | 검색을 위한 쿼리 문                                                             |
| form/size | 조회 갯수 ( from : 시작, size :크기 ), \[ 기본 ( form:0, size:n ) 으로 from 생략 가능 ] |
| sort      | 검색 결과가 \_score가 아닌 필드 기준으로 정렬                                           |
| \_source  | 결과 중 특정 필드의 내용만 출력                                                      |
| highlight | 결과 중 검색어와 매칭 되는 단어 강조                                                   |
| boost     | 결과를 스코어로 변경                                                             |
| scroll    | n개의 단위로 나뉘어서                                                            |

## 1. 주소 필드에 부산이 포함된 검색

```
curl --location --request GET 'https://172.19.164.132:9200/tourlist/_search?pretty=null' \
--header 'Content-Type: application/json' \ 
--data '{
    "query": {
        "term": {
            "addr1": "부산"
        }
    }
}'
```

## 2. 한번에 가지올 크기 지정 : from: size:

응답에 있는 hits.total.value 에 검색된 갯수기 있습니다.

```
curl --location --request GET 'https://172.19.164.132:9200/tourlist/_search?pretty=true' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic ZWxhc3RpYzoxbj1lQmFzTTJqR3gxRVpGMDdEWQ==' \
--data '{
    "size": 3,
    "query": {
        "term": {
            "addr1": "부산"
        }
    }
}'

결과
{
    "took": 9,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 5,
            "relation": "eq"
        },
        "max_score": 0.83591366,
        "hits": [
            {
                "_index": "tourlist",
                "_id": "A4StQYoB7JUIma-S4U-_",
                "_score": 0.83591366,
                "_source": {
                    "_class": "com.hyomee.service.elastic.tour.doc.TourListDoc",
                    "contentid": "2726843",
                    "addr1": "부산광역시 강서구 천성동",
                    ....
                }
            },
            {
                "_index": "tourlist",
                "_id": "AoStQYoB7JUIma-S4U8s",
                "_score": 0.83591366,
                "_source": {
                    "_class": "com.hyomee.service.elastic.tour.doc.TourListDoc",
                    "contentid": "129156",
                    "addr1": "부산광역시 강서구 외양포로 10",
                    ...                    
                }
            },
            {
                "_index": "tourlist",
                "_id": "BoStQYoB7JUIma-S408r",
                "_score": 0.79428625,
                "_source": {
                    "_class": "com.hyomee.service.elastic.tour.doc.TourListDoc",
                    "contentid": "2782772",
                    "addr1": "부산광역시 강서구 거가대로 2571",
                    ....
                }
            }
        ]
    }
}
```

## 3. 정렬&#x20;

필드명을 기준으로 정렬 : desc, asc

```
curl --location --request GET 'https://localhost:9200/tourlist/_search?pretty=true' \
--header 'Content-Type: application/json' \ 
--data '{
    "size": 3,
    "sort": {
        "cat1.keyword": "desc",
        "createdtime.keyword": "desc"        
    },
    "query": {
        "term": {
            "addr1": "부산"
        }
    }
}'
```

## 4 원하는 필드만 가져오기

주소 필드에 부산이 포함된 결과 중 title과 overview만 가져 오기

```
curl --location --request GET 'https://172.19.164.132:9200/tourlist/_search?pretty=null' \
--header 'Content-Type: application/json' \ 
--data '{
    "_source": ["title" , "overview"],
    "query": {
        "term": {
            "addr1": "부산"
        }
    }
}'
결과
{
    "took": 212,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 5,
            "relation": "eq"
        },
        "max_score": 0.83591366,
        "hits": [
            {
                "_index": "tourlist",
                "_id": "A4StQYoB7JUIma-S4U-_",
                "_score": 0.83591366,
                "_source": {
                    "title": "가덕도 연대봉",
                    "overview": "예로부터 더덕이 많이 나서 붙여졌다는 이름인 가덕도에 위치한 가장 높은 곳이다. 가덕도는 부산에 속한 섬 중 가장 큰 섬으로 신석기 시대부터 사람이 살았던 것으로 추정되며, 임진왜란과 일제강점기까지 우리나라의 아린 역사를 지켜본 섬이다.<br /> \n연대봉은 해발 459미터의 높이로 2~3시간이면 다녀올 수 있는 산으로 높이가 그리 높지도 않아 오르기도 어렵지 않지만 정상에 오르면 높은 산에 오른 듯한 기분과 전망을 경험할 수 있다. 산세가 원만하여 가족단위로 가볍게 등산하기에 알맞으며 정상에 가까이 오르면 가덕도 바다와 거가대교를 한 눈에 볼 수 있다.<br />\n연대봉 정상에는 임진왜란 당시 최초로 왜적을 발견해 불을 피워 올렸던 봉수대를 볼 수 있다.<br />\n산과 바다의 청취를 동시에 느낄 수 있는 곳이다."
                }
            },
            {
                "_index": "tourlist",
                "_id": "AoStQYoB7JUIma-S4U8s",
                "_score": 0.83591366,
                "_source": {
                    "title": "가덕도 등대",
                    "overview": "* 고대역사를 간직하고 있는 가덕도 등대 *<br />옛 등대 시설은 사무실과 숙소, 등탑이 연결된 복합건물 형태로써 중앙에 높이 9.2m의 등탑이 세워져 있으며, 붉은 벽돌과 미송을 사용했고 출입구 천정에는 그 당시 황실의 상징인 오얏꽃 모양의 문장이 새겨져 있으며, 함석으로 된 지붕은 부식방지를 위해 함석 위에 피치타르가 덮혀져 있다. 가덕도는 한반도의 동남단에 자리잡아 동으로는 사하구 다대포, 서남북은 거제도 동북바다, 북으로는 유라시아와 환태평양의 관문이면서 동북아 허브항만으로 건설한 부산항신항(2006.1.19)과 접하며 진해시 용원동과 접하며 진해시 용원과의 거리는 4㎞, 인근의 거제도와의 거리는 10㎞ 정도이다.<br /><br />* 가덕도는 해상교류의 요츙지 *<br />가덕도가 해상교류 및 군사적으로 중요시 된 것은 조선 중종 39년(1544년) 가덕진과 천성진을 설치하여 임진왜란 때는 치열한 격전장이기도 했던 곳이었으며, 현재 진해만으로 들어가는 중요 요충지로서 가덕도등대는 대한제국의 말기인 1909년 12월에 건립되었으며 옛 가덕도 등대건물은 서구 건축양식으로 지어진 건축물의 하나로 2003년 9월 부산시 유형문화재로 지정되었다. 또한 해양수산부에서도 영구보존 시설로 지정해 원형을 보존하고 있다. 2002년 새로 지어진 지금의 등대는 팔각으로 된 돌출형이며 등탑높이는 40.5m로 우리나라에서 두 번째로 높은 등대이다.<br /><br />* 최초점등일 - 1909년 12월 <br />* 구조 - 백팔각 콘크리트(40m)<br />* 등질 - 섬백광 12초 1섬광(FlW12s) <br />* 특징 - 낙동강 하구언에 위치하며 조선시대부터 봉화를 올려 뱃길을 안내하였던 연대봉 등 고대역사를 간직한 섬으로 등대체험학교를 운영하고 있다. <br>"
                }
            },
            {
                "_index": "tourlist",
                "_id": "BoStQYoB7JUIma-S408r",
                "_score": 0.79428625,
                "_source": {
                    "title": "가덕해양파크휴게소",
                    "overview": "부산~거제 간 연결 도로의 휴게소로 천혜의 자연을 가득 담은 풍경 맛집이다. 부산의 서쪽 끝 가덕도에 있어 드넓은 야외테라스와 힐링 산책로에서 광활한 바다 풍경을 즐길 수 있는 곳이다. 어린왕자 등 여러 가지 조형물과 벚꽃나무 군락이 있어 사진 찍기 좋은 포인트가 곳곳에 마련되어 있고, 뒤편에는 맘껏 뛰어놀 수 있는 넓은 광장과 먼바다 풍경을 조망할 수 있는 전망대도 있다. 부산~거제 간 연결 도로의 핵심인 가덕해저터널에 대한 모든 것이 전시된 세계 최대수심 해저터널 연결도로 홍보전시관도 볼거리다."
                }
            },
            {
                "_index": "tourlist",
                "_id": "BIStQYoB7JUIma-S4k8u",
                "_score": 0.75660795,
                "_source": {
                    "title": "가덕도대항인공동굴",
                    "overview": "2차 세계대전 말에 일본군은 전력이 급강하하자 부산과 주변 해안에 미군 상륙작전을 대비하기 위한 방어시설을 구축하였는데 가덕도 대항마을의 인공동굴도 이와 같은 정세에서 조성된 것으로 보인다. 가덕도 대항 인공동굴은 현재 10여 개 정도 발견되었으며 탄광 노동자들을 데려와 만들었다고 전해진다. 형태는 I자, T자, L자형 동굴과 연결 복식 동굴이 있고, 내부 통로가 십자형으로 얽혀 있을 정도로 긴 동굴도 있다. 동굴 내부에 당시 상황을 이해하기 쉽게 조형물로 표현해 놓아 가슴 아픈 역사의 흔적을 되짚어 볼 수 장소이다."
                }
            },
            {
                "_index": "tourlist",
                "_id": "AYStQYoB7JUIma-S4E_A",
                "_score": 0.75660795,
                "_source": {
                    "title": "가덕도",
                    "overview": "가덕도는 부산 최대의 섬으로 자동차로 여행하기 좋은 곳이다. 최고봉인 연대산을 비롯해 수많은 산들과 해안가를 따라 이어진 힐링 둘레길, 일제강점기의 역사 유적지 등 볼거리가 많다. 특히 아름다운 일출, 일몰과 사계절 월척이 가능한 낚시 포인트는 최고의 자랑거리이다. \n\n* 일출&일몰&낚시&차박 캠핑 포인트: \n  - 대항새바지(일출)\n  - 새바지등대(일출)\n  - 연대봉(일출/일몰)\n  - 가덕도등대(일출)\n  - 천성항(일몰/낚시)\n  - 대항전망대(일몰)\n  - 대항방파제(낚시)\n  - 새바지방파제(낚시) \n  - 외양포항(일몰/차박 캠핑)\n  - 동선새바지(일출/일몰/차박 캠핑)"
                }
            }
        ]
    }
}
```

&#x20;

참고 :  [4.4 검색 API - \_search API - Elastic 가이드북 (kimjmin.net)](https://esbook.kimjmin.net/04-data/4.4-\_search)&#x20;

참고 : [https://www.elastic.co/guide/en/elasticsearch/reference/7.17/search-your-data.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/search-your-data.html)

&#x20;

&#x20;
