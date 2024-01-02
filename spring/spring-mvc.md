# Spring MVC

## 1. @Controller

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

## 2. @RestController

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

## 3. Spring MVC – 사용하는 Annotation

## &#x20;3-1. @RequestBody, @ResponseBody

* @RequestBody : Http요청의 Body내용을 자바 객체로 매핑&#x20;
* @ResponseBody : 자바 객체를 Http 응답의 Body내용으로 매핑 ( Json )

```java
@RestController
Public class ServiceController {   
   // Http 요청의 내용을 RequestDTO 객체에 매핑하기 위해서 @RequestBody 사용
   // @ResponseBody를 사용 하지 않는 이유 : @RestController 사용 하였기 때문
   // @Controller를 사용 하는 경우 :  @ResponseBody를 사용 해야 함 
   @RequestMapping(value=“/uri/process”, method = RequestMethod.POST)
   public ResponseDTO process(@RequestBody RequestDTO requestDTO) {      ResponseDTO responseDTO = service.process(requestDTO);
      return responseDTO;   
```

### 3-2. @RequestMapping Method

* GET : 요청 받은 URI의 정보를 검색&#x20;
* POST : 요청된 자원을 생성&#x20;
* PUT : 요청된 자원을 수정 , 요청된 자원 전체 갱신
* PATCH : 요청된 자원을 수정, 일부만 갱신&#x20;
* DELETE : 요청된 자원 삭제 OPTIONS : 지원 되는 메소드의 종류 확인

#### 3-2-1. Method 수준

```java
@RestController
Public class ServiceController {   
   @RequestMapping(value=“/uri/process”, method = RequestMethod.POST)
   public ResponseDTO process(@RequestBody RequestDTO requestDTO) { … }
     
   // 복수 설정 
   @RequestMapping(value={“/uri/process”, “/uri/process01”},                               method = RequestMethod.POST)
   public ResponseDTO process(@RequestBody RequestDTO requestDTO) { … }
}

```

#### 3-2-2. Class 수준

```java
@RestController@RequestMapping(value=“/uri/process”)
Public class ServiceController {   
    // Uri : /uri/process  -> Http Request Method만 사용
   // @RequestMethod.POST, GET, PUT, PATCH, DELETE, TRACE, OPTIONS   
   @RequestMapping(method = RequestMethod.POST)
   public ResponseDTO process(@RequestBody RequestDTO requestDTO) { … }
   
   // Uri : /uri/process/processA  
   @RequestMapping(value=“/processA”,                              
                   method = RequestMethod.POST)
   public ResponseDTO process(@RequestBody RequestDTO requestDTO) { … }
   
   // 복수 설정
   @RequestMapping(value={“/processA”, “/processB”},                               
                                    method = RequestMethod.POST)
   public ResponseDTO process(@RequestBody RequestDTO requestDTO) { … }

}

```
