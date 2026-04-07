# Xxl Job


<!--more-->
## 1 xxl-job

### 1.1 xxl-job 是什么
是分布式任务调度平台
特点：
- 支持分布式集群部署
- 可视化界面管理任务
- 故障转移
- 日志
- 支持分片执行
### 1.2 为什么需要xxl-job
springboot任务调用
- springboot的定时任务是单机部署的，在微服务场景，每个服务都会执行定时任务，造成重复执行
- 无日志
- 不知道任务执行情况
- 每次修改需要重启服务
- 宕机任务停止

xxl-job解决了这些问题

### 1.3 核心架构

两个角色：
- xxl-job-admin(调度中心) ， 负责：提供web界面，触发任务，记录日志，告警等
- executor（执行器）： 你的程序中定义的任务

```text
┌─────────────────────────────────────────────────────────┐
│                    调度中心 (Admin)                       │
│   - 提供 Web 管理界面                                     │
│   - 负责触发任务、记录日志、发送报警                        │
│   - 统一管理所有执行器和任务                               │
└──────────────────────┬──────────────────────────────────┘
                       │ HTTP 通信
         ┌─────────────┴─────────────┐
         ▼                           ▼
┌─────────────────┐       ┌─────────────────┐
│  执行器 A        │       │  执行器 B        │
│ (你的业务服务)   │       │ (你的业务服务)   │
│  @XxlJob 方法   │       │  @XxlJob 方法   │
└─────────────────┘       └─────────────────┘
```


### 1.4 快速开始

#### 1.4.1 1.docker 部署xxl-job-admin

```java
docker run -e PARAMS=" --spring.datasource.url=jdbc:mysql://host.docker.internal:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai --spring.datasource.username=root --spring.datasource.password=123456 --xxl.job.accessToken=root " -p 8080:8080 --name xxl-job-admin xuxueli/xxl-job-admin:2.4.2
```

#### 1.4.2 2.引入依赖
```xml
<dependency>  
    <groupId>com.xuxueli</groupId>  
    <artifactId>xxl-job-core</artifactId>  
    <version>3.1.1</version>  
</dependency>
```
#### 1.4.3 3.配置文件

```yml
xxl:  
  job:  
    executor:  
      admin:  
        addresses: http://localhost:8080/xxl-job-admin  # 调度中心地址
      appname: demo-executor   # 执行器名称，要和调度中心注册的一致
      ip:  # 留空，自动获取本机IP
      port: 9999  # 执行器端口，集群部署时每台机器不同
      logPath: /data/xxl-job/log  # 日志存储路径
      logRetentionDays: 30  # 日志保留天数
      accessToken: root     # 和调度中心的 token 保持一致 也就是上面docker的参数里的
```

#### 1.4.4 4.配置类
```java
  
@Configuration  
public class XxlJobConfig {  
  
    @Value("${xxl.job.executor.admin.addresses}")  
    private String adminAddresses;  
  
    @Value("${xxl.job.executor.appname}")  
    private String appname;  
  
    @Value("${xxl.job.executor.ip:}")  
    private String ip;  
  
    @Value("${xxl.job.executor.port:9999}")  
    private int port;  
  
    @Value("${xxl.job.executor.accessToken:}")  
    private String accessToken;  
  
    @Value("${xxl.job.executor.logPath:}")  
    private String logPath;  
  
    @Value("${xxl.job.executor.logRetentionDays:30}")  
    private int logRetentionDays;  
  
  //注册xxljob执行器
    @Bean  
    public XxlJobSpringExecutor xxlJobExecutor() {  
        XxlJobSpringExecutor executor = new XxlJobSpringExecutor();  
        executor.setAdminAddresses(adminAddresses);  
        executor.setAppname(appname);  
        executor.setIp(ip);  
        executor.setPort(port);  
        executor.setAccessToken(accessToken);  
        executor.setLogPath(logPath);  
        executor.setLogRetentionDays(logRetentionDays);  
        return executor;  
    }  
}
```

#### 1.4.5 你的业务逻辑(定时任务)

```java
@Component  
public class DemoJobHandler {  
  
  /**  
 * @XxlJob("value") 中的 value 就是在调度中心配置的 JobHandler 名称  
 */
    @XxlJob("demoJobHandler")       
    public void demoJobHandler() {  
        System.out.println("XXL-JOB running...");  
    }  
}
```


#### 1.4.6 6.启动项目打开web页面

启动项目后，打开`http://localhost:8080/xxl-job-admin/`

打开右侧执行器管理
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260322164129089.png)


![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260322164252446.png)
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260322164313472.png)

点击启动后就开始进行定时任务了

---

> 作者: cheems  
> URL: http://localhost:59658/java/springboot/b335cec9/  

