# 1.全局跨域配置类

```java
package cn.udream.saas.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

/**
 * @author fengzhiqiang
 * @date 2021/8/27 10:06
 */
@Configuration
public class GlobalCorsConfig {
    @Bean
    public CorsFilter corsFilter() {
        // 1.添加CORS配置信息
        CorsConfiguration config = new CorsConfiguration();
        // 2.放行哪些原始域
        config.addAllowedOrigin("*");
        // 3.是否发送Cookie信息
        config.setAllowCredentials(true);
        // 4.放行哪些原始域(请求方式)
        config.addAllowedMethod("OPTIONS");
        config.addAllowedMethod("HEAD");
        config.addAllowedMethod("GET");     // get
        config.addAllowedMethod("PUT");     // put
        config.addAllowedMethod("POST");    // post
        config.addAllowedMethod("DELETE");  // delete
        config.addAllowedMethod("PATCH");
        config.addAllowedHeader("*");

        // 5.添加映射路径
        UrlBasedCorsConfigurationSource configSource = new UrlBasedCorsConfigurationSource();
        configSource.registerCorsConfiguration("/**", config);

        // 6.返回新的CorsFilter.
        return new CorsFilter(configSource);
    }
}
```



