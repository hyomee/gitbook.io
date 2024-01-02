# Compound queries

여러 유형의 쿼리를 복합적으로 사용할 때 사용 하는 쿼리로 다음과 같은 유형이 있습니다.

* bool : 항목 내 쿼리에 일치하는 항목이 있을 때 검색
* boosting : 검색 결과의 가중치를 높이기 위한 검색
* constant\_score :  filter query를 감싸서 매칭되는 모든 문서에 대해 boost parameter value와 동일한 relevance score를 반환합니다. 이를 통해 검색 결과의 정확도를 높일 수 있습니다
* dis\_max : 루씬의 DisjunctionMaxQuery에서 따온 용어로 예를 들어 멀티 키워드 검색을 수행 시 여러 필드에서 검색을 수행하게 되는 조건에서 멀티 키워드의 키워드와 동일한 키워드의 score를 더욱 높이 평가 하는 방식
* function\_score : 검색 결과의 정확도를 높이기 위해 사용하며, 이를 통해 검색 결과의 relevance score를 조정하고, 특정 필드의 중요도를 높이거나 낮출 수 있습니다.

자세한 학습은 참고 사이트을 활용하고 여기에서는 bool query에 대해서 간단히 기술합니다.

참고 : [https://www.elastic.co/guide/en/elasticsearch/reference/7.17/compound-queries.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/compound-queries.html)
