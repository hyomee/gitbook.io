# Filter

## 1. servlet.Filter

javax.servlet-api나 tomcat-embed-core를 사용하면 제공되는 servlet filter interface

```java
@Order(1)
@Component
public class ServletFilter implements Filter {  
  @Override
  public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
          throws IOException, ServletException {     
     chain.doFilter(request, response); 
   }
}

```

## 2. Spring에서 제공 하는 GenericFilterBean

```java
@Order(3)
public class GenericFilterBeanFlter extends GenericFilterBean { 
   @Override
   public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
          throws IOException, ServletException {
      // TODO 전처리 
      chain.doFilter(request, response); 
   }
}

@Configuration
public class GenericFilterBeanFilterConfig {
   @Bean
    public GenericFilterBeanFlter GenericFilterBeanFlte() {
       return new GenericFilterBeanFlter();
    }
}

```

