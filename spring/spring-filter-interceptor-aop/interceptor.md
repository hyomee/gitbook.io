# Interceptor

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

     @Autowired
     private SomeInterceptor someInterceptor;
     
     @Override
     public void addInterceptors(InterceptorRegistry registry) {
          registry.addInterceptor(someInterceptor)
               .addPathPatterns("/some/**")
               .excludePathPatterns("/etc/**");
          }
}

```

```java
@Component
public class SomeInterceptor extends HandlerInterceptorAdapter {

    Logger logger = LoggerFactory.getLogger(SomeInterceptor.class);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler){
logger.info("===== before(interceptor) =====");
        return true;
    }
    
    @Override
    public void postHandle(
            HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)
            throws Exception {
    logger.info("===== after(interceptor) =====");
    }
 
    @Override
    public void afterCompletion(
            HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
            throws Exception {
    logger.info("===== afterCompletion =====");
    }
}

```
