# Pageable

**실제 운영 환경에서는 Pageable 속성을 지정하여 메모리 아웃을 피해야 합니다.**&#x20;

## **1.  도메인 클래스 생성**

```
@Setter
@Getter
@Table(name = "TB_DEMO")
@AllArgsConstructor
@NoArgsConstructor
@Entity
@Builder
public class DemoEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private int id;    
    private String title;
    private String content;
}
```

## **2. Repository 생성**

페이징 및 정렬을 하기 위해서는 ListPagingAndSortingRepository 또는 PagingAndSortingRepository를 사용 합니다.

* 확장받을 수 있는 Repository는 다음과 같습니다.
  * CrudRepository  :  CRUD 기능을 제공을 전체 기능을 사용하지 않을 때 사용&#x20;
  * PagingAndSortingRepository :  페이지 및 정렬을 수행하는 메서드를 제공&#x20;
  * ListPagingAndSortingRepository : PagingAndSortingRepository 를 상속받아List를 반환 하는 메서드 젝공\
    \- Page\<T>, Slice\<T>에 있는 페이징 관련 정보 없이 데이터 만 받을 수 있습니다.\
    \- 전체 페이지(getTotalPages()), 전체 갯수(getTotalElements())를 제공 받지 못 합니다.
  * JpaRepository는 지속성 컨텍스트 플러시 및 일괄처리의 레코드 삭제와 같은 JPA 관련 메소드를 제공 \
    \- CrudRepository 및 PagingAndSortingRepository의 전체 API가 포함 되어 있습니다.

<figure><img src="https://blog.kakaocdn.net/dn/c1QHKF/btsr0WvMLRN/2DgOV0j1iv8SNXuRAF7npk/img.png" alt=""><figcaption><p>JpaRespositoty 상속 구조</p></figcaption></figure>

다음 예제는 JpaRepository를 상속 받아 List와 Page의 차이점에서 알아보고자 합니다.

```
@Repository
public interface TourlistRepository extends  JpaRepository<TourlistEntity, String > {
  List<TourlistEntity> findByTitleContains(String title, Pageable pageable);
  Page<TourlistEntity> findTourlistEntitiesByTitleContains(String title, Pageable pageable);
}
```

\


## &#x20;**3. 서비스 생성**&#x20;

```
@RequiredArgsConstructor
@Service
public class TourListService {

  private final TourlistRepository tourlistRepository;

  /**
   * ListPagingAndSortingRepository를 사용해서 Repository에서 List로 반환 값 사용
   * @param title
   * @param pageable
   * @return
   */
  public List<TourlistDTO> getTourList(String title, Pageable pageable) {
     List<TourlistEntity> tourlistEntities = tourlistRepository.findByTitleContains(title, pageable);
     return WorkMapper.INSTANCE.toTourlistDTOs(tourlistEntities);
  }

  /**
   * PagingAndSortingRepository를 사용해서 Repository에서 Page로 반환 값 사용
   * Page<T>, Slice<T> 페이징 관련 속성 사용 
   * @param title
   * @param pageable
   * @return
   */
  public ResponsePageDTO getTourListPage(String title, Pageable pageable) {
    Page<TourlistEntity> tourlistEntityPage = tourlistRepository.findTourlistEntitiesByTitleContains(title, pageable);
    return ResponsePageDTO.setResponsePageDTO(tourlistEntityPage);
  }
}
```

ResponsePageDTO 객체는 전체 페이지 등 페이지 정보 및 데이터를 담고 있는 객체로 ResponsePageDTO.setResponsePageDTO를 통해서 생성할 됩니다. setResponsePageDTO의 소스는 다음과 같습니다.

```
@Builder
public class ResponsePageDTO {
  public PageDTO page;
  public Object content;

  public static ResponsePageDTO setResponsePageDTO(Page<?> page) {

    return ResponsePageDTO.builder()
            .page(PageDTO.builder()
                    .total( page.getTotalElements())                // 조회된 전체 갯수
                    .pageCount(page.getTotalPages())                // 조회된 전체 페이지 갯수
                    .pageNumber(page.getPageable().getPageNumber()) // 현재 페이지
                    .pageSize(page.getPageable().getPageSize())     // 페이지에 있는 컨텐츠 갯수
                    .build())
            .content(page.getContent())                             // 컨텐츠
            .build();

  }
}
```

## **4. Controller 생성**

```
@RestController
@RequestMapping("/tourlist")
public class TourlistController {

  private final TourListService tourListService;

  @GetMapping("/{title}")
  public ResponseEntity getTourlist(@PathVariable String title,
                                    Pageable pageable) {
    return ResponseUtils.completed(tourListService.getTourList(title, pageable));
  }

  @GetMapping("/page/{title}")
  public ResponseEntity getTourlistPage(@PathVariable String title,
                                        @PageableDefault(page=0, size=10)Pageable pageable) {
    return ResponseUtils.completed(tourListService.getTourListPage(title, pageable));
  }
}
```

### 4-1. RestApi :&#x20;

http://localhost:8080/tourlist/{title}?page=0\&size=2

* Method : get
* PathVariable :
  * title : 검색 조건
  * page : 현재페이지
  * size : 페이지당 개수
* 기능 : 조건에 맞는 것을 처음부터 페이지당 개수만큼 조회하는 것으로 내부적으로는 PageRequest.of(page, pageSize, sort)를 사용합니다.
  * 예제는 sort 조건이 없는 것입니다, page는 0부터 시작하는 것에 주의를 해야 합니다.
  * page : 0, size : 10 이면 처음부터 10개를 의미하고
  * page : 1, size : 처음 + 1부터 10개를 의미합니다.&#x20;
* 결과 :

```
{
    "contentid": "127480",
    "title": "가거도(소흑산도)",
    ....
    "overview": "가거도는 중국의 새벽닭 울음소리가 들릴만큼 중국땅과 가깝다는 ...."
    ....
},
{
    "contentid": "129139",
    "title": "신안 가거도 등대",
    ....
    "overview": "가거도(소흑산도)등대는 중국 상하 ....",
    ....
}
```

### 4-2. RestApi&#x20;

&#x20;http://localhost:8080/tourlist/page/{title}?page=0\&size=2

* Method : get
* PathVariable :
  * title : 검색 조건
  * page : 현재페이지
  * size : 페이지당 개수
* 기능 : 조건에 맞는 것을 처음부터 페이지당 개수만큼 조회하는 것으로 내부적으로는 PageRequest.of(page, pageSize, sort)를 사용하며 페이지 정보를 포함해서 반환합니다.

```
"page": {
            "total": 382,
            "pageNumber": 0,
            "pageCount": 191,
            "pageSize": 2
        },
        "content": [
            {
                "contentid": "127480",
                "title": "가거도(소흑산도)",
                .....
                "overview": "가거도는 중국의 새벽닭 울음소리가 ...",
                ....
            },
            {
                "contentid": "126273",
                "title": "가계해변",
                ....
                "overview": "바닷물이 갈라지는 현대판 모세의 ...",
                ....
            }
        ]
    }
```

\*\* page, size를 입력하지 않고  "http://localhost:8080/tourlist/page/가" 로 호출 이면 @PageableDefault(page=0, size=10) 로 설정 한 값을 설정이 됩니다.

\*\* Pageable은 org.springframework.data.web 패키지의 안에 정의된 ArgumentResolver인 PageableHandlerMethodArgumentResolver에 정의되어 있습니다.

```java
public class PageableHandlerMethodArgumentResolver extends PageableHandlerMethodArgumentResolverSupport
		implements PageableArgumentResolver {
        ...
}
```

## **5. Pageable의 속성**

Pageable의 기본 속성을 정의하는 방법은 다음과 같이 3 가지 방법이 있습니다.\
&#x20;PageRequest.of(page, pageSize, sort) -> PageRequest(int page, int size, Sort sort)

### 5-1. applicatpn.yml 에 설정하는 방법&#x20;

```java
@ConfigurationProperties("spring.data.web")
public class SpringDataWebProperties {
	... 
}
```

에 정의되어 있으며 applicaion.yml에 다음과 같이 정의합니다.

```yaml
  data:
    web:
      pageable:
        default-page-size: 10
        max-page-size: 30
```

### 5-2. bean 등록하는 방법

```java
@Configuration
public class CustomPageableConfiguration {
    @Bean
    public PageableHandlerMethodArgumentResolverCustomizer customize() {
        return p -> p.setFallbackPageable(PageRequest.of(0, 50));
    }
}
```

spring-boot-starter-web의 SpringDataWebConfiguration Class에 정의되어 있으면 위 소스와 같이 별도로 구성 개발 하지 않으면 SpringDataWebAutoConfiguration Class에 정의된 PageableHandlerMethodArgumentResolverCustomizer  메서드에 의해서 기본 설정이 됩니다, ( applicaion.yml  읽음 )

기본 속성 정의는 @PageableDefault ->  CustomPageableConfiguration bean -> applicaion.yml ->등록 순서로 적용이 됩니다.

## **6. Sort**&#x20;

다음과 같이 정렬할 수 있습니다.

```java
Sort sort1 = Sort.by("name").descending();   // 이름 기준 내림차순
Sort sort2 = Sort.by("name").ascending();    // 이름 기준 오름차순


Pageable sortedByName = 
  PageRequest.of(0, 3, Sort.by("name"));

Pageable sortedByPriceDesc = 
  PageRequest.of(0, 3, Sort.by("name").descending());

// 이름과 나이로 정렬
Pageable sortedByPriceDescNameAsc = 
  PageRequest.of(0, 5, Sort.by("name").descending().and(Sort.by("age")));
```

http로 sort 하는 방법&#x20;

```bash
단건 :
http://localhost:8080/tourlist/가?page=0&size=2&sort=contentid,desc

다중 :
http://localhost:8080/tourlist/가?page=0&size=2&sort=contentid,desc&sort=areacode,asc
```

\
