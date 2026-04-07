# 事件机制


<!--more-->
## 1 简述

### 1.1 定义
基于观察者模式，允许组件之间进行通信的一种机制

### 1.2 用途
- 用于异步处理： 发送邮件，日志记录
- 应用启动或关闭通知

### 1.3 核心概念

主要是三个类：
- ApplicationEvent  所有事件的基类，表示一个应用事件
- ApplicationListener： 事件监听器
- ApplicationEventPublisher： 事件发布器

流程：
- **定义事件**（可选继承 `ApplicationEvent`）
- **发布事件**（通过 `ApplicationEventPublisher`）
- **监听事件**（实现 `ApplicationListener` 或使用 `@EventListener` 注解）
- **处理事件**（监听器中的逻辑）

发布事件 (ApplicationEventPublisher)
          │
          ▼
Spring 事件广播器 (EventMulticaster)
          │
          ▼
调用对应的 ApplicationListener / @EventListener 方法

### 1.4 发送邮件示例

#### 1.4.1 定义事件
```java
  
/**  
 * 用户事件 —— 事件对象（Event）  
 *  
 * <p>在 Spring 事件机制中，事件对象承载着「发生了什么」的所有信息。  
 * 就像一封信，里面写清楚了事情的来龙去脉，监听者收到后按需处理。  
 *  
 * <p>继承 {@link ApplicationEvent} 是 Spring 事件体系的要求：  
 *   - source：事件来源（谁触发了这个事件，通常传 this）  
 *   - timestamp：事件发生时间戳（父类自动记录）  
 */  
@Getter  
public class UserEvent extends ApplicationEvent {  
  
    /** 新注册用户的用户名 */  
    private final String userName;  
  
    /** 新注册用户的邮箱地址（监听器发送欢迎邮件时会用到） */  
    private final String email;  
  
    /**  
     * 构造用户事件  
     *  
     * @param source   事件发布者（通常是发布事件的 Service，传 this 即可）  
     * @param userName 用户名  
     * @param email    用户邮箱  
     */  
    public UserEvent(Object source, String userName, String email) {  
        super(source);  
        this.userName = userName;  
        this.email = email;  
    }  
}
```

#### 1.4.2 发布事件

```java
/**  
 * 用户服务 —— 事件发布者（Publisher）  
 *  
 * <p>核心思路：UserService 只专注于「创建用户」这一件事。  
 * 创建完成后，通过 {@link ApplicationEventPublisher} 发布一个 {@link UserEvent} 事件。  
 * 至于后续要做什么（发邮件、发短信、记日志等），UserService 完全不关心，  
 * 由各自的监听器独立处理 —— 这就是事件机制带来的「解耦」效果。  
 *  
 * <p>{@link ApplicationEventPublisher} 是 Spring 容器提供的事件总线，  
 * 调用 publishEvent() 后，Spring 会找到所有监听该事件类型的监听器并通知它们。  
 */  
@Slf4j  
@RequiredArgsConstructor  
@Service  
public class UserService {  
  
    /**  
     * Spring 事件发布器  
     * Spring Boot 会自动将其注入，无需任何额外配置  
     */  
    private final ApplicationEventPublisher publisher;  
  
    /**  
     * 创建用户  
     *  
     * <p>业务流程：  
     *   1. 执行用户创建逻辑（此处模拟，实际项目中会写入数据库）  
     *   2. 发布 UserEvent 事件，通知所有关心「用户创建」的监听器  
     *  
     * <p>注意：publishEvent() 默认是「同步」的，即在当前线程依次调用所有监听器。  
     * 若监听器方法上标注了 @Async，则该监听器会异步执行，不阻塞当前线程。  
     *  
     * @param userName 用户名  
     * @param email    用户邮箱  
     */  
    public void createUser(String userName, String email) {  
        // 第一步：执行核心业务逻辑（实际项目中这里会调用 Repository 写入数据库）  
        log.info("[用户服务] 用户创建成功 -> 用户名: {}, 邮箱: {}", userName, email);  
  
        // 第二步：发布事件  
        // 构造事件对象，将用户信息封装进去，供监听器使用  
        // this 表示事件来源是当前 UserService 实例  
        UserEvent event = new UserEvent(this, userName, email);  
        publisher.publishEvent(event);  
  
        log.info("[用户服务] 已发布 UserEvent 事件，等待监听器处理...");  
        // 注意：由于监听器是 @Async 异步执行的，这里「发布完即返回」，不会等邮件发完  
    }  
}
```

#### 1.4.3 监听事件

```java
  
/**  
 * 用户事件监听器 —— 事件监听者（Listener）  
 *  
 * <p>Spring 事件机制的三个核心角色：  
 *   1. 事件（Event）          ：{@link UserEvent}，描述「发生了什么」  
 *   2. 发布者（Publisher）    ：{@link com.example.demo.service.UserService}，触发事件  
 *   3. 监听者（Listener）     ：本类，对事件作出响应  
 *  
 * <p>这种设计的好处（解耦）：  
 *   - UserService 只负责创建用户和发布事件，完全不知道「发邮件」这件事  
 *   - 如果将来要加短信通知，只需新增一个监听器，无需修改 UserService  
 *   - 符合「开闭原则」：对扩展开放，对修改关闭  
 */  
@Slf4j  
@RequiredArgsConstructor  
@Component  
public class UserEventListener {  
  
    /** 注入邮件服务，用于发送欢迎邮件 */  
    private final EmailService emailService;  
  
    /**  
     * 处理用户注册事件，发送欢迎邮件  
     *  
     * <p>@EventListener：告诉 Spring「当 UserEvent 事件发布时，调用这个方法」  
     *   Spring 通过方法参数类型来匹配事件，类型匹配则自动调用。  
     *  
     * <p>@Async：异步执行（重要！）  
     *   - 不加 @Async：发邮件和创建用户在「同一个线程」里顺序执行  
     *     → 发邮件慢（网络IO）会阻塞接口响应，用户等待时间变长  
     *   - 加 @Async：发邮件在「独立线程池」里执行  
     *     → 用户注册接口立即返回，邮件在后台异步发送，体验更好  
     *   注意：@Async 需要在启动类上加 @EnableAsync 才能生效  
     *  
     * @param event 用户事件，包含用户名和邮箱等信息  
     */  
    @Async                // 异步执行，不阻塞主线程  
    @EventListener        // 声明这是一个事件监听方法  
    public void onUserCreated(UserEvent event) {  
        log.info("[事件监听] 收到用户注册事件 -> 用户: {}, 邮箱: {}",  
                event.getUserName(), event.getEmail());  
  
        // 调用邮件服务发送欢迎邮件  
        // 此处在异步线程中执行，即使发送失败也不会影响用户注册流程  
        emailService.sendWelcomeEmail(event.getEmail(), event.getUserName());  
    }  
}
```


#### 1.4.4 业务逻辑

```java
/**  
 * 邮件服务 —— 负责邮件的组装与发送  
 *  
 * <p>企业中通常将邮件发送单独封装为一个 Service，原因：  
 *   1. 单一职责：业务 Service 只负责业务逻辑，不关心邮件细节  
 *   2. 复用性：多个地方需要发邮件时，直接调用该 Service  
 *   3. 可测试：可以单独对邮件服务做单元测试或 Mock  
 * * <p>{@link JavaMailSender} 是 Spring 对 JavaMail API 的封装，  
 * 配置好 application.yaml 后，Spring Boot 会自动注入该 Bean。  
 */  
@Slf4j                  // Lombok：自动生成 log 变量，用于打印日志  
@RequiredArgsConstructor // Lombok：自动生成包含所有 final 字段的构造器（用于依赖注入）  
@Service  
public class EmailService {  
  
    /**  
     * Spring Boot 自动配置并注入的邮件发送器  
     * 底层封装了 SMTP 连接、认证、发送等所有细节  
     */  
    private final JavaMailSender mailSender;  
  
    /**  
     * 从配置文件读取发件人地址  
     * 对应 application.yaml 中的 spring.mail.username  
     */    @Value("${spring.mail.username}")  
    private String from;  
  
    /**  
     * 发送 HTML 格式的欢迎邮件  
     *  
     * <p>企业中大多数通知邮件使用 HTML 格式，因为可以加样式、图片、链接，体验更好。  
     * {@link MimeMessage} 支持 HTML 内容，而简单的 {@link org.springframework.mail.SimpleMailMessage} 只支持纯文本。  
     *  
     * @param to       收件人邮箱  
     * @param userName 用户名（用于邮件正文个性化）  
     */  
    public void sendWelcomeEmail(String to, String userName) {  
        try {  
            // 1. 创建 MIME 邮件对象（支持 HTML、附件、内嵌图片）  
            MimeMessage message = mailSender.createMimeMessage();  
  
            // 2. 使用 Helper 辅助类设置邮件属性  
            //    第二个参数 true 表示开启 multipart 模式（支持 HTML 和附件）  
            MimeMessageHelper helper = new MimeMessageHelper(message, true, "UTF-8");  
  
            helper.setFrom(from);                          // 发件人  
            helper.setTo(to);                              // 收件人  
            helper.setSubject("欢迎注册！");                // 邮件主题  
            helper.setText(buildHtmlContent(userName), true); // 邮件正文（true=HTML格式）  
  
            // 3. 发送邮件  
            mailSender.send(message);  
  
            log.info("[邮件服务] 欢迎邮件发送成功 -> 收件人: {}", to);  
  
        } catch (Exception e) {  
            // 企业中邮件发送失败通常不应影响主业务流程，只记录日志并告警  
            // 可根据业务需要决定是否重试（结合消息队列实现可靠投递）  
            log.error("[邮件服务] 欢迎邮件发送失败 -> 收件人: {}, 原因: {}", to, e.getMessage(), e);  
        }  
    }  
  
    /**  
     * 构建 HTML 邮件正文  
     *  
     * <p>企业中通常使用模板引擎（如 Thymeleaf、FreeMarker）渲染邮件模板，  
     * 这里为了演示简单，直接拼接 HTML 字符串。  
     *  
     * @param userName 用户名  
     * @return HTML 字符串  
     */  
    private String buildHtmlContent(String userName) {  
        return "<div style='font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto;'>" +  
               "  <h2 style='color: #4A90E2;'>欢迎加入我们！</h2>" +  
               "  <p>亲爱的 <strong>" + userName + "</strong>，</p>" +  
               "  <p>您已成功注册，欢迎使用我们的平台！</p>" +  
               "  <p style='color: #888; font-size: 12px;'>此邮件由系统自动发送，请勿回复。</p>" +  
               "</div>";  
    }  
}
```


### 1.5 实现原理

发布事件 -> 调用事件广播器 

广播器遍历监听器 -> 匹配类型 ->  调用监听方法

异步/顺序通过线程池和 @order来控制


### 1.6 与消息队列的区别


事件机制是在jvm内的轻量的异步/同步通知机制 ，不持久化，不会重试，不会消息确认，消息有可能丢失。  简单，复杂度低，由线程池控制

消息队列： 跨进程/系统，消息可靠性搞，持久化。

---

> 作者: cheems  
> URL: http://localhost:59658/java/springboot/3c6e54ea/  

