# SpringBoot Mybatis 구성

Mybatis는  SQL, 저장 프로시저를 자바 오브젝트에 매핑하는 것을 지원하는 SQL 매퍼 프레임워크 입니다.

참고 : [MyBatis – 마이바티스 3 | 소개](https://mybatis.org/mybatis-3/ko/index.html)

## 1. 스프링 부트에서 MyBatis를 적용&#x20;

### **1.  의존성 설정**&#x20;

mybatis-spring-boot-starter는 mybatis-spring-boot-autoconfigure, spring-boot-starter, spring-boot-starter-jdbc, mybatis-spring 의존성을 포함하고 있습니다.

```
# Maven
<dependency>
  <groupId>org.mybatis.spring.boot</groupId>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <version>3.0.2</version>
</dependency>

# Gradle
implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:3.0.2'
```

\-  [Spring Boot H2 ](https://hyomee.tistory.com/101)사용하기를 통해서 데이터 베이스를 연결 해야 합니다.

### **2.  Application.yml 설정**

```
mybatis:
  mapper-locations: config/mybatis/mapper/**/*.xml
  configuration:
    map-underscore-to-camel-case: true
```

&#x20;\- mapper-locations : myBatis 쿼리 \
&#x20;\- configuration : [myBatis  설정](https://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/)  &#x20;

### **3. 테스트 코드 작성**

#### 3-1. 데이터 객체 생성 - mybatis를 통해서 쿼리 실행 후 데이터를 받을 객체

```
@Setter
@Getter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@ToString
public class DemoDTO {
  private int id;
  private String name;
}
```

#### 3-2 Mapper Interface 생성

Mapper Query 와 연결 되는 부분으로 mapper.xml 파일의 mapper namespacem, id 속성과 일치 하여야 합니다,

```
package com.xxx.mapper

import org.apache.ibatis.annotations.Mapper;

@Mapper
// xml : mapper namespace="com.xxx.mapper.MybatisDemoMapper"
public interface MybatisDemoMapper { 

  // select id="retrieveDemo" 
  List<DemoDTO> retrieveDemo();

}
```

#### 3-3 Mapper Query XML 생성

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.hyomee.hybird.mybatis.demo.mapper.MybatisDemoMapper">
    <select id="retrieveDemo" resultType="com.hyomee.hybird.mybatis.demo.dto.DemoDTO">
      SELECT ID,
             NAME
        FROM TB_DEMO
    </select>
</mapper>
```

기본적으로 1,2,3 번 항목 까지 작성이 되었으면 SpringBoot를 이용 하여 진행이 됩니다.\
Bean 등록을 통해서 해야 하는 경우가 있으면 다음을 진행 합니다.

### **4. Bean 등록으로 Mybatis 연결**

#### 4-1. @Configuration,  @MapperScan을 Mapper 스캔&#x20;

```java
@Configuration
@MapperScan(basePackages = {"com.xxx.web.**.service. **.mapper"}, 
       sqlSessionFactoryRef="sqlSessionFactory", sqlSessionTemplateRef="sqlSessionTemplate")
public class MyBatisConfig {


  @Bean(name="sqlSessionFactory")
  public SqlSessionFactory sqlSessionFactory(@Qualifier("dataSource") DataSource dataSource, 
         ApplicationContext applicationContext) throws Exception {

    SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
    sqlSessionFactoryBean.setDataSource(dataSource);
    sqlSessionFactoryBean.setTypeAliasesPackage("com.xxx.web.**.dto");
    sqlSessionFactoryBean.setConfigLocation(applicationContext.getResource("classpath:mybatis-config.xml"));
    sqlSessionFactoryBean.setMapperLocations(applicationContext.getResources("classpath:/mapper/**/*.xml"));
    return sqlSessionFactoryBean.getObject();
  }

  @Bean(name="sqlSessionTemplate")
  public SqlSessionTemplate sqlSessionTemplate(@Qualifier("sqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
    return new SqlSessionTemplate(sqlSessionFactory);
  }
}
```

\- Mybatis 설정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <setting name="callSettersOnNulls" value="true"/>
        <setting name="jdbcTypeForNull" value="NULL"/>
    </settings>
</configuration>
```

&#x20;

#### 4-2. 초기 스크립트 생성

```java
@Configuration
public class PersistenceConfig {

  @Bean
  public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.H2)
        .addScript("schema.sql")
        .addScript("data.sql")
        .build();
  }

  @Bean
  public SqlSessionFactory sqlSessionFactory() throws Exception {
    SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
    factoryBean.setDataSource(dataSource());
    return factoryBean.getObject();
  }
}
```
