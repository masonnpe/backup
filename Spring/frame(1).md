Mybatis缓存

- 一级缓存是SqlSession级别的缓存，缓存的数据只在SqlSession内有效,在操作数据库的时候需要先创建SqlSession会话对象，在对象中有一个HashMap用于存储缓存数据，此HashMap是当前会话对象私有的

- 二级缓存是mapper级别的缓存，同一个namespace公用这一个缓存，所以对SqlSession是共享的；使用 LRU 机制清理缓存，通过 cacheEnabled 参数开启。  1.当一个sqlseesion执行了一次select后，在关闭此session的时候，会将查询结果缓存到二级缓存

  　　　　　　2.当另一个sqlsession执行select时，首先会在他自己的一级缓存中找，如果没找到，就回去二级缓存中找，找到了就返回，就不用去数据库了，从而减少了数据库压力提高了性能　

## Sqlsessionfactorybean

实现了FactoryBean和InitializingBean接口

FactoryBean 一旦某个bean实现次接口，那么通过getbean方法获取bean时，其实时获取此类的getObject()返回的实例

InitializingBean 会在初始化时调用其afterPropertiesSet方法来进行bean的逻辑初始化

## MapperFactoryBean

MapperFactoryBean初始化 ，在DaoSupport类中实现  checkDaoConfig  对Dao配置的验证 initDao  对dao的初始工作，initdao时模板方法













```java
public class AuthIntercepter implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        HandlerMethod method=(HandlerMethod)o;
        UserAuth userAuth = AnnotationUtils.findAnnotation(method.getMethod(), UserAuth.class);
        if(userAuth.permit().equals(httpServletRequest.getParameter("token"))){
            return true;
        }
        return false;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {

    }
}


@Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface UserAuth {

    String permit() default "";
}

@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new AuthIntercepter()).addPathPatterns("/**");
    }
}
```

