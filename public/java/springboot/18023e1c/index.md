# 后端跨域处理


<!--more-->
## 1 三种方式

### 1.1 @CrossOrigin
在单个控制器/接口上加这个注解

```java
@RestController
@RequestMapping("/api")
@CrossOrigin(origins = "http://localhost:3000")
public class UserController {

    @GetMapping("/users")
    public String users() {
        return "ok";
    }
}
```


### 1.2 全局配置WebMvcConfigurer(推荐)
```java
@Configuration  
public class CorsConfig implements WebMvcConfigurer {  
  
  
    @Override  
    public void addCorsMappings(CorsRegistry registry) {  
  
        registry.addMapping("/**")  
                .allowedOrigins("*")  
                .allowedMethods("*")  
                .allowedHeaders("*")  
                .allowCredentials(true)  
                .maxAge(3600);  
        WebMvcConfigurer.super.addCorsMappings(registry);  
    }  
}
```


### 1.3 CorsFilter

CorsFilter 更加底层一点

```java
@Bean  
public CorsFilter corsFilter() {  
    CorsConfiguration config = new CorsConfiguration();  
    config.setAllowCredentials(true);  
    config.addAllowedOriginPattern("*");  
    config.addAllowedHeader("*");  
    config.addAllowedMethod("*");  
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();  
    source.registerCorsConfiguration("/**", config);  
  
    return new CorsFilter(source);  
  
}
```


>如果使用了spring security，还需要放行

```java
@Bean  
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {  
		http  
		.cors(cors -> {})  
		.csrf(csrf -> csrf.disable())  
		.authorizeHttpRequests(auth -> auth  
		      .anyRequest().permitAll()  
		);  
  
return http.build();  
}
```

---

> 作者: cheems  
> URL: http://localhost:59658/java/springboot/18023e1c/  

