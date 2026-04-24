# Springboot拦截器与过滤器

拦截
<!--more-->
## 1 拦截器 - interceptor
### 1.1 什么是拦截器 -what
Spring Boot 中的拦截器是基于 Spring MVC 的一种请求增强机制，在请求到达controller前后，插入自定义逻辑

### 1.2 用途

- 日志
- 权限校验，拦截
- 统计接口耗时

### 1.3 执行流程

请求到达controller之前，先经过拦截器的三个部分：
- preHandle  :    在controller执行之前调用
- postHandle ： 在controller执行之后，视图渲染之前。 前后端分离项目一般不用
- afterCompletion： 在整个请求完成之后调用
<br/>
执行顺序：
```text
preHandle
   ↓
Controller
   ↓
postHandle
   ↓
afterCompletion
```

<br/>
如果有多个拦截器
<br/>
按注册顺序执行 `preHandle`
<br/>
按相反顺序执行 `postHandle` 和 `afterCompletion`
<br/>
比如注册顺序是：
<br/>
- InterceptorA
- InterceptorB
<br/>
<br/>
执行顺序就是：
A.preHandle  
B.preHandle  
Controller  
B.postHandle  
A.postHandle  
B.afterCompletion  
A.afterCompletion



### 1.4 具体使用 --- 日志案例
步骤：
1. 新建一个拦截器类LogInterceptor，实现HandlerInterceptor接口，将其交给ioc管理（@Component）
2. 重写preHandle方法， 内部写前置拦截的逻辑
3. 新建配置类 WebConfig ，实现WebMvcConfigurer接口，同样给ioc容器管理（@Configuration）
4. 重写addInterceptors方法，将LogInterceptor注入进来，并在方法内部registry.addInterceptor(logInterceptor);

#### 1.4.1 日志案例

> 前置拦截preHandle

<br/>
从请求中获取 
1. 方法名
2. uri
3. 方法请求参数
4. ip
5. 当前用户
6. 添加一个当前时间参数
通过日志打印出来

>afterCompletion

<br/>
获取请求状态，uri
以及计算请求花费时间


```java
  
@Slf4j  
@Component  
public class LogInterceptor implements HandlerInterceptor {  
  
    private static final String START_TIME_ATTR = "requestStartTime";  
  
    @Override  
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {  
        long startTime = System.currentTimeMillis();  
        request.setAttribute(START_TIME_ATTR, startTime);  
  
        String method = request.getMethod();  
        String uri = request.getRequestURI();  
        String queryString = request.getQueryString();  
        String ip = getClientIp(request);  
  
        SessionUser sessionUser = (SessionUser) request.getSession().getAttribute(UserConstant.USER);  
        String userInfo = sessionUser != null  
                ? "userId=" + sessionUser.getId() + ", role=" + sessionUser.getUserRole()  
                : "未登录";  
  
        log.info("[请求开始] {} {} {} | IP: {} | 用户: {}",  
                method, uri, queryString != null ? "?" + queryString : "", ip, userInfo);  
  
        return true;  
    }  
  
    @Override  
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {  
        // 控制器执行后、视图渲染前，REST 项目一般无需处理  
    }  
  
    @Override  
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {  
        Long startTime = (Long) request.getAttribute(START_TIME_ATTR);  
        long cost = startTime != null ? System.currentTimeMillis() - startTime : -1;  
  
        int status = response.getStatus();  
        String uri = request.getRequestURI();  
  
        if (ex != null) {  
            log.error("[请求异常] {} | 状态码: {} | 耗时: {}ms | 异常: {}", uri, status, cost, ex.getMessage());  
        } else {  
            log.info("[请求结束] {} | 状态码: {} | 耗时: {}ms", uri, status, cost);  
        }  
    }  
  
    private String getClientIp(HttpServletRequest request) {  
        String ip = request.getHeader("X-Forwarded-For");  
        if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {  
            ip = request.getHeader("X-Real-IP");  
        }  
        if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {  
            ip = request.getRemoteAddr();  
        }  
        // X-Forwarded-For 可能包含多个 IP，取第一个  
        if (ip != null && ip.contains(",")) {  
            ip = ip.split(",")[0].trim();  
        }  
        // IPv6 本地回环地址转为 IPv4        if ("0:0:0:0:0:0:0:1".equals(ip) || "::1".equals(ip)) {  
            ip = "127.0.0.1";  
        }  
        return ip;  
    }  
}
```

注册拦截器
```java
@Configuration  
public class WebConfig implements WebMvcConfigurer {  
  
  
    @Resource  
    private LogInterceptor logInterceptor;  
  
  
    public void addInterceptors(InterceptorRegistry registry) {  
    registry.addInterceptor(logInterceptor)  
            .addPathPatterns("/**")  
            .excludePathPatterns("/static/**");  
}
  
}
```


### 1.5 与其他拦截方式的区别

#### 1.5.1 filter 过滤器
filter 属于servlet规范，更底层，在spring外也可以用

interceptor属于spring mvc机制，更加贴近业务

执行顺序： Filter -> Interceptor -> Controller

#### 1.5.2 AOP 

interceptor 更加针对web请求

AOP 针对 方法进行拦截

## 2 过滤器 -filter
### 2.1 什么是过滤器
是servlet规范提供的机制， 用于在请求到达 spring mvc 之前 对请求进行预处理

请求来了 → 过滤器 → 拦截器 → Controller → 返回 → 拦截器 → 过滤器


### 2.2 用途

- **字符编码**：统一设置请求和响应编码
- **跨域处理**：CORS 配置
- **日志记录**：请求日志、耗时统计
- **安全检查**：IP 黑名单、简单权限
- **请求包装**：包装 request/response 修改数据

### 2.3 示例

#### 2.3.1 定义一个filter
继承Filter 类
加上@WebFilter(urlPatterns = "/*") 注解  拦截所有请求
```java
import jakarta.servlet.*;
import jakarta.servlet.http.HttpServletRequest;
import java.io.IOException;

public class MyFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("过滤器初始化");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        System.out.println("请求进入过滤器：" + req.getRequestURI());

        // 放行
        chain.doFilter(request, response);

        System.out.println("响应返回过滤器");
    }

    @Override
    public void destroy() {
        System.out.println("过滤器销毁");
    }
}
```
### 2.4 注册

#### 2.4.1 方式一： 直接在过滤器上加上@Componet注解

### 2.5 方式二：使用 `FilterRegistrationBean`（推荐，可以精确控制，要过滤哪些请求）
```java
@Configuration  
public class FilterConfig {  
  
    @Bean  
    public FilterRegistrationBean<MyFilter> filterRegistrationBean() {  
        FilterRegistrationBean<MyFilter> myFilterFilterRegistrationBean = new FilterRegistrationBean<>(new MyFilter());  
        myFilterFilterRegistrationBean.addUrlPatterns("/*"); //拦截所有请求  
        myFilterFilterRegistrationBean.setName("MyFilter");  
        myFilterFilterRegistrationBean.setOrder(1); //数字越小优先级越高  
        return myFilterFilterRegistrationBean;  
    }  
  
  
}
```

### 2.6 方式三：`@WebFilter` + `@ServletComponentScan`
```java

  
@WebFilter(urlPatterns = "/*")  
//@Component  
public class MyFilter implements Filter {  
  
    @Override  
    public void init(FilterConfig filterConfig) throws ServletException {  
        System.out.println("过滤器初始化");  
    }  
  
    @Override  
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)  
            throws IOException, ServletException {  
        HttpServletRequest req = (HttpServletRequest) request;  
        System.out.println("请求进入过滤器：" + req.getRequestURI());  
  
        // 放行  
        chain.doFilter(request, response);  
  
        System.out.println("响应返回过滤器");  
    }  
  
    @Override  
    public void destroy() {  
        System.out.println("过滤器销毁");  
    }  
}
```

```java
  
@ServletComponentScan  
@SpringBootApplication  
public class DemoApplication {  
  
    public static void main(String[] args) {  
        SpringApplication.run(DemoApplication.class, args);  
    }  
  
}
```


### 2.7 记录请求耗时 实战 


#### 2.7.1 1.定义一个过滤器类


继承Filter 类
加上@WebFilter(urlPatterns = "/*") 注解  拦截所有请求
```java
public class TimeFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        long start = System.currentTimeMillis();
        HttpServletRequest req = (HttpServletRequest) request;

        chain.doFilter(request, response);

        long end = System.currentTimeMillis();
        System.out.println(req.getRequestURI() + " 耗时：" + (end - start) + " ms");
    }
}
```


#### 2.7.2 2.springboot中注册过滤器

##### 2.7.2.1 方法一

加@WebFilter注解在filter上

```java
@SpringBootApplication
@ServletComponentScan // 扫描@WebFilter注解
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

##### 2.7.2.2 方法二
这种不需要@WebFilter注解
```java
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean<CorsFilter> corsFilter() {
        FilterRegistrationBean<CorsFilter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(new CorsFilter());
        registrationBean.addUrlPatterns("/*"); // 拦截路径
        registrationBean.setOrder(1);          // 执行顺序
        return registrationBean;
    }
}
```

##### 2.7.2.3 方式三：加@Component注解

---

> 作者: cheems  
> URL: http://localhost:1313/java/springboot/a667c40c/  

