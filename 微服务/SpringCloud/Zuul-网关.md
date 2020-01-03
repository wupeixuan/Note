## 路由

在启动类上加@EnableZuulProxy


在url输入http://127.0.0.1:8085/product/product/list

第一个product为erueka中的名称。


自定义路由
```
zuul:
  routes:
    # /myProduct/product/list -> /product/product/list
    myProduct:
      path: /myProduct/**
      serviceId: product
    # 简洁写法
    product: /myProduct/**
  # 排除某些路由
  ignored-patterns:
    - /**/product/listForOrder
```

## 限流

## 权限校验

在前置过滤器中实现相关逻辑

## 跨域

- 在被调用的类或者方法上加@CrossOrigin
- 在Zuul里增加CorsFilter过滤器

```
import org.springframework.web.filter.CorsFilter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import java.util.Arrays;

@Configuration
public class CorsConfig {
    @Bean
    public CorsFilter corsFilter() {
        final UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        final CorsConfiguration config = new CorsConfiguration();

        config.setAllowCredentials(true);
        config.setAllowedOrigins(Arrays.asList("*"));
        config.setAllowedHeaders(Arrays.asList("*"));
        config.setAllowedMethods(Arrays.asList("*"));
        config.setMaxAge(300L);

        source.registerCorsConfiguration("/**", config);
        return new CorsFilter(source);
    }
}
```