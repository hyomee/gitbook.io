# JPA -  페이징

## 문제점&#x20;

스프링 데이터 JPA를 사용해서 페이징 처리를 하는 방법

## 해결방안

Spring Data JPA를 사용해서 페이징은 원리만 파악하면 간단히 해결이 가능합니다.

제공되는 Pageable 속성을 사용하면 되는데 주의 할 사항은 속성을 지정해서 전체 데이터가 로딩 되지 않도록 해야 한다는 것입니다.&#x20;

[<mark style="color:purple;">**자세한 방법은 JPA Pagination에서 확인 할 수 있습니다.**</mark>](https://hyomee.gitbook.io/develop/spring-data/spring-data-jpa/pageable)
