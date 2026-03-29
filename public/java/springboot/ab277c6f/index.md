# Springboot实现异步处理


<!--more-->
异步处理：将方法的执行提交到一个独立的线程中运行，调用方线程不等待结果，继续往下执行。

## 1 基础方式：@Async注解

### 1.1 步骤

#### 1.1.1 开启异步支持
在配置类上添加@EnableAsync  

```java
@EnableAsync  
@SpringBootApplication  
public class DemoApplication {  
  
    public static void main(String[] args) {  
        SpringApplication.run(DemoApplication.class, args);  
    }  
  
}
```
#### 1.1.2 在方法上添加@Async

异步方法必须是 public
```java
  @Async
    public void asyncMethodVoid() {
        System.out.println("Start async task...");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Async task finished");
    }

    @Async
    public CompletableFuture<String> asyncMethodWithResult() {
        System.out.println("Start async task with result...");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Async task with result finished");
        return CompletableFuture.completedFuture("Hello, async!");
    }
```

#### 1.1.3 调用
```java
@RequiredArgsConstructor  
@RequestMapping("/async")  
@RestController  
public class AsyncController {  
  
    private final AsyncService asyncService;  
  
  
    @GetMapping("")  
    public String hello() {  
        CompletableFuture<String> stringCompletableFuture = asyncService.asyncMethodWithResult();  
        return stringCompletableFuture.join();  
    }  
  
    @GetMapping("/a")  
    public void a() {  
        asyncService.asyncMethodVoid();  
    }  
  
}
```

### 1.2 使用自定义线程池

默认 `@Async` 使用 **SimpleAsyncTaskExecutor**，它每次都会创建新线程,生产环境一般自定义线程池

```java
@Slf4j  
@EnableAsync  
@Configuration  
public class AsyncConfig implements AsyncConfigurer {  
  
    @Nullable  
    @Override    public Executor getAsyncExecutor() {  
  
        ThreadPoolTaskExecutor threadPoolTaskExecutor = new ThreadPoolTaskExecutor();  
        threadPoolTaskExecutor.setCorePoolSize(5);  
        threadPoolTaskExecutor.setMaxPoolSize(10);  
        threadPoolTaskExecutor.setQueueCapacity(500);  
        threadPoolTaskExecutor.setThreadNamePrefix("Async-");  
        threadPoolTaskExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());  
        threadPoolTaskExecutor.initialize();  
        return threadPoolTaskExecutor;  
  
    }  
  
    @Nullable  
    @Override    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {  
  
        return (exception, method, objects) -> {  
            log.info("异步方法异常：{}，原因: {}", method.getName(), exception.getMessage() );  
        };  
    }  
}
```















---

> 作者: cheems  
> URL: http://localhost:1313/java/springboot/ab277c6f/  

