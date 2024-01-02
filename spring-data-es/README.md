---
description: Spring Data Elasticsearch 5.1.3 버젼을 중심으로 작성된 문서 입니다.
---

# Spring-Data-ES

Spring Data의 하위 프로젝트 중 하나로, Elasticsearch 검색 엔진과의 통합을 제공한다. POJO 중심의 모델과 리포지토리 스타일의 데이터 액세스 레이어를 쉽게 작성할 수 있습니다.

## **Spring Data Elasticsearch의 기능**

Spring 설정 지원, ElasticsearchTemplate 헬퍼 클래스, 객체 매핑, 리포지토리 인터페이스의 자동 구현 등을 제공합니다.

* Spring Data Elasticsearch는 단일 Elasticsearch 노드 또는 클러스터에 연결된 Elasticsearch 클라이언트에서 작동합니다.
* Elasticsearch Client를 사용하여 클러스터 작업을 수행 할 수 있지만 Spring Data Elasticsearch를 사용하는 애플리케이션은 일반적으로 상위 레벨의 Elasticsearch Operations 및 Elasticsearch Repositories를 사용합니다.

**Spring Data 사이트에 있는 Features .**

* Spring Data Elasticsearch는 Java 기반의 @Configuration 클래스 또는 ES 클라이언트 인스턴스를 위한 XML 네임스페이스를 사용하여 Spring 구성을 지원한다.
* &#x20;ElasticsearchTemplate 헬퍼 클래스는 일반적인 ES 작업을 수행하여 생산성을 높일수.있으며 POJO 간의 통합된 객체 매핑을 포함한다.&#x20;
* Spring의 Conversion Service와 통합된 기능이 풍부한 객체 매핑을 제공한다.&#x20;
* 어노테이션 기반 매핑 메타데이터를 지원하며, 다른 메타데이터 형식을 지원할 수 있도록 확장 가능하다.&#x20;
* Repository 인터페이스의 자동 구현과 사용자 정의 finder 메서드를 지원합니다.&#x20;
* CDI는 Repository에 대한 지원을 제공한다.

자세한 기능에 대해서는[ Spring Data Elasticsearch 사이트](https://spring.io/projects/spring-data-elasticsearch)에서 제공하는 가이드를 참조하면.됩니다.



