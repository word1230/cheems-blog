---
date: 2026-03-21T22:01:08+08:00
draft: false
slug: bf39541e
type: posts
title: Springboot集成jwt
collections: 项目
categories:
  - java
---

<!--more-->


## jwt简介

### 什么是jwt

是用于信息传递的令牌，常用于身份认证和授权
<br/>
特点：
- **自包含**：token 内部携带用户信息（Claims），服务器无需频繁查询数据库。
- **可验证性**：token 可以被签名，防止被篡改。
- **无状态**：服务器不需要存储 session（常用于分布式系统）。

### 为什么需要jwt
- 在微服务项目中，需要分布式session才能保证多服务器session一致。这样每次请求都要去redis读取，同时管理session变得复杂
- 高并发情况下，session存储压力大，需要额外考虑过期和清理策略
- 跨域不方便

jwt的好处：
- 无状态： jwt自带用户身份信息，服务端不需要存储，减轻压力。 同时可扩展性强
- 跨域/平台传递： json格式，与前后端语言无关
- 自包含： 载荷包含用户信息，不需要查数据库

解决了
- 不依赖服务器 Session
- 不依赖 Cookie 跨域限制
- 可以轻松扩展到微服务


### jwt 的构成

三个部分，中间用 `.` 隔开
`header.payload.signature`

#### 头部

声明算法和类型
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

被base64编码 后 -> header

#### 负载

存放 **声明（Claims）**，即 token 的信息
常见字段：
**注册声明（Registered Claims）**：
- `iss`：签发者
- `sub`：主题（一般是用户 id）
- `aud`：受众
- `exp`：过期时间（秒）
- `nbf`：生效时间
- `iat`：签发时间
- `jti`：JWT ID（唯一标识）
自定义字段 role ，username

同样会被base64编码

### 签名

签名用于**验证 token 的完整性和真实性**：
```text
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secretKey
)
```

- 服务器拿到 token 时，可以使用相同的密钥验证签名是否一致。
- 如果 token 被篡改，签名校验会失败。


### jwt 使用流程
- 用户登录 → 服务端验证账号密码 → 返回 JWT
- 客户端保存 JWT（一般在 localStorage 或 Cookie）
- 客户端每次请求 → 在请求头 `Authorization` 带上 JWT
- 服务端拦截器 / Filter 验证 JWT
    - 合法 → 放行，解析用户信息
    - 不合法 → 返回 401
- Controller 处理请求 → 返回响应



## 集成步骤

### 1. 引入依赖

主要是两部分的依赖：

- jwt相关依赖
```xml
  
<!-- jjwt-api：JWT 的 Java 实现，用于生成、解析、验证 JWT Token --><dependency>  
    <groupId>io.jsonwebtoken</groupId>  
    <artifactId>jjwt-api</artifactId>  
    <version>0.12.6</version>  
</dependency>  
<!-- jjwt 运行时实现 -->  
<dependency>  
    <groupId>io.jsonwebtoken</groupId>  
    <artifactId>jjwt-impl</artifactId>  
    <version>0.12.6</version>  
    <scope>runtime</scope>  
</dependency>  
<!-- jjwt JSON 序列化支持 -->  
<dependency>  
    <groupId>io.jsonwebtoken</groupId>  
    <artifactId>jjwt-jackson</artifactId>  
    <version>0.12.6</version>  
    <scope>runtime</scope>  
</dependency>

```
<br/>
- spring security
<br/>
```java
<!-- Spring Security：提供认证与授权框架，JWT 过滤器会整合进这里 -->  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-security</artifactId>  
</dependency>
```

同时还可以添加 mybatis-plus mysql 等数据库相关依赖

### 2.写jwt工具类

工具类里主要做三件事：
- 生成token
- 检验token
- 从token中解析我们存入的数据


#### 生成token

需要设置：主题，载荷，签发时间，过期时间，密钥

```java

/**  
 * JWT 工具类 —— 核心！  
 *  
 * ===================== 什么是 JWT？ =====================  
 * JWT（JSON Web Token）是一种用于在各方之间安全传递信息的开放标准（RFC 7519）。  
 * 它是一个字符串，由三部分组成，用「.」分隔：  
 *  
 *   Header.Payload.Signature * *   1. Header（头部）：描述算法类型  
 *      { "alg": "HS256", "typ": "JWT" }  
 *      → Base64 编码 → eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9  
 * *   2. Payload（载荷）：存放用户信息（不要放敏感数据！）  
 *      { "sub": "1", "username": "admin", "role": "ADMIN", "iat": 1700000000, "exp": 1700086400 }  
 *      → Base64 编码 → eyJzdWIiOiIxIiwidXNlcm5hbWUiOiJhZG1pbiJ9  
 * *   3. Signature（签名）：防篡改验证  
 *      HMACSHA256(base64(Header) + "." + base64(Payload), 密钥)  
 *      → 只有持有密钥的服务器才能验证签名是否正确  
 *  
 * ===================== JWT 的工作流程 =====================  
 *  登录：用户提交用户名密码 → 服务器验证 → 生成 JWT → 返回给客户端  
 *  请求：客户端每次请求携带 JWT → 服务器验证 JWT → 解析用户信息 → 处理业务  
 *  
 * ===================== JWT 的优点 =====================  
 *  - 无状态：服务器不需要存储 Session，适合分布式系统  
 *  - 自包含：Token 本身携带用户信息，无需查数据库验证身份  
 *  - 跨域友好：通过 HTTP Header 传递，不受 Cookie 跨域限制  
 *  
 * @Component 让 Spring 管理这个 Bean，可以在其他地方用 @Autowired 注入  
 */
@Component
public class JwtUtil {

    /**
     * JWT 签名密钥
     * @Value 从 application.yaml 的 jwt.secret 读取配置值
     */
    @Value("${jwt.secret}")
    private String secret;

    /**
     * Token 过期时间（毫秒）
     * @Value 从 application.yaml 的 jwt.expiration 读取配置值
     */
    @Value("${jwt.expiration}")
    private long expiration;

    /**
     * 获取签名用的 SecretKey 对象
     * 将字符串密钥转换为 HMAC-SHA 算法所需的 Key 对象
     */
    private SecretKey getSigningKey() {
        // 将密钥字符串转为字节数组，再生成 SecretKey
        return Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8));
    }

    // ==================== 生成 Token ====================

    /**
     * 生成 JWT Token
     *
     * @param userId   用户 ID
     * @param username 用户名
     * @param role     角色
     * @return JWT Token 字符串，例如：eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIn0.xxx
     */
    public String generateToken(Long userId, String username, String role) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + expiration);

        return Jwts.builder()
                // sub（Subject）：主题，通常存用户 ID
                .subject(String.valueOf(userId))
                // 自定义载荷字段：存储用户名和角色
                .claim("username", username)
                .claim("role", role)
                // iat（Issued At）：签发时间
                .issuedAt(now)
                // exp（Expiration）：过期时间
                .expiration(expiryDate)
                // 用密钥签名，指定算法 HS256
                .signWith(getSigningKey())
                // 生成最终的 Token 字符串
                .compact();
    }
4m}
```

#### 解析token

从token中解析出我们设置的载荷信息，同时解析的过程中也对token进行了验证

```java
public Claims parseToken(String token) {
        return Jwts.parser()
                // 设置验证签名用的密钥（必须与生成时相同）
                .verifyWith(getSigningKey())
                .build()
                // 解析 Token，同时验证签名和过期时间
                .parseSignedClaims(token)
                // 获取载荷部分
                .getPayload();
    }
    
    /**
     * 从 Token 中提取用户 ID
     *
     * @param token JWT Token 字符串
     * @return 用户 ID
     */
    public Long getUserId(String token) {
        // getSubject() 获取 sub 字段的值（我们存的是用户 ID 字符串）
        return Long.parseLong(parseToken(token).getSubject());
    }

    /**
     * 从 Token 中提取用户名
     *
     * @param token JWT Token 字符串
     * @return 用户名
     */
    public String getUsername(String token) {
        return parseToken(token).get("username", String.class);
    }

    /**
     * 从 Token 中提取角色
     *
     * @param token JWT Token 字符串
     * @return 角色字符串，如 "ADMIN" 或 "USER"
     */
    public String getRole(String token) {
        return parseToken(token).get("role", String.class);
    }
```



#### 验证token
token能顺利解析，就说明token是有效的
```java
/**
     * 验证 Token 是否有效
     *
     * 有效条件：
     *  1. Token 格式正确（不是乱码）
     *  2. 签名验证通过（没有被篡改）
     *  3. Token 没有过期
     *
     * @param token JWT Token 字符串
     * @return true = 有效，false = 无效
     */
    public boolean validateToken(String token) {
        try {
            parseToken(token);
            return true;
        } catch (ExpiredJwtException e) {
            // Token 已过期
            System.out.println("Token 已过期: " + e.getMessage());
        } catch (MalformedJwtException e) {
            // Token 格式错误（不是合法的 JWT）
            System.out.println("Token 格式错误: " + e.getMessage());
        } catch (UnsupportedJwtException e) {
            // 不支持的 Token 类型
            System.out.println("不支持的 Token 类型: " + e.getMessage());
        } catch (Exception e) {
            // 其他错误（如签名验证失败）
            System.out.println("Token 验证失败: " + e.getMessage());
        }
        return false;
    }
```


### 3. 写jwt认证过滤器

对token进行验证
生成认证信息



```java
/**
 * JWT 认证过滤器 —— 整个项目的安全核心！
 *
 * ===================== 工作原理 =====================
 * 每一个 HTTP 请求到达服务器时，都会先经过这个过滤器：
 *
 *  请求 → [JwtAuthenticationFilter] → Controller → 响应
 *
 * 过滤器做以下事情：
 *  1. 从请求 Header 中读取 Token
 *     Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
 *  2. 验证 Token 是否有效（格式、签名、是否过期）
 *  3. 如果有效，解析出用户信息，存入 Spring Security 上下文
 *  4. 后续 Controller 可以从上下文中获取当前登录用户
 *
 * ===================== OncePerRequestFilter =====================
 * 继承这个类保证每次请求只执行一次过滤，避免重复处理。
 *
 * @Component 注册为 Spring Bean，在 SecurityConfig 中会被添加到过滤器链
 */
@Component
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtUtil jwtUtil;

    /**
     * 过滤器核心方法，每个请求都会执行
     *
     * @param request     HTTP 请求
     * @param response    HTTP 响应
     * @param filterChain 过滤器链（调用 chain.doFilter 才能让请求继续传递）
     */
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {

        // 第一步：从请求 Header 中提取 Token
        String token = extractTokenFromRequest(request);

        // 第二步：如果 Token 存在且有效，进行认证
        if (StringUtils.hasText(token) && jwtUtil.validateToken(token)) {

            // 从 Token 中解析用户信息（不需要查数据库！这就是 JWT 无状态的优势）
            String username = jwtUtil.getUsername(token);
            String role = jwtUtil.getRole(token);
            Long userId = jwtUtil.getUserId(token);

            // 构建 Spring Security 的权限列表
            // "ROLE_" 前缀是 Spring Security 的约定，角色必须加这个前缀
            List<SimpleGrantedAuthority> authorities = List.of(
                    new SimpleGrantedAuthority("ROLE_" + role)
            );

            // 创建认证对象
            // UsernamePasswordAuthenticationToken 是 Spring Security 的标准认证令牌
            // 参数：(用户标识, 凭证, 权限列表)
            // 这里第一个参数传 username，凭证传 null（Token 验证过了，不需要密码）
            UsernamePasswordAuthenticationToken authentication =
                    new UsernamePasswordAuthenticationToken(username, null, authorities);

            // 将认证信息存入 SecurityContext（安全上下文）
            // 之后在 Controller 中可以通过以下方式获取当前用户：
            //   String currentUser = (String) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }

        // 第三步：放行请求，继续传递给下一个过滤器或 Controller
        // 无论 Token 是否有效，都要调用这个方法，否则请求会卡住
        filterChain.doFilter(request, response);
    }

    /**
     * 从 HTTP 请求中提取 JWT Token
     *
     * Token 存放在请求头 Authorization 中，格式为：
     *   Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIn0.xxx
     *
     * 我们需要去掉 "Bearer " 前缀，只取后面的 Token 字符串。
     *
     * @param request HTTP 请求
     * @return Token 字符串，如果不存在或格式不对则返回 null
     */
    private String extractTokenFromRequest(HttpServletRequest request) {
        // 读取 Authorization 请求头
        String bearerToken = request.getHeader("Authorization");

        // 检查是否存在且以 "Bearer " 开头
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
            // 截取 "Bearer " (7个字符) 之后的部分，即真正的 Token
            return bearerToken.substring(7);
        }

        return null;
    }
}
```

### 4. 写SecurityConfig类


配置 
- 哪些请求要有什么权限
- 异常情况：如果无权限返回什么
- 注册过滤器

```java
/**
 * Spring Security 配置类 —— 安全规则定义中心
 *
 * ===================== Spring Security 简介 =====================
 * Spring Security 是 Spring 生态的安全框架，默认会拦截所有请求并要求登录。
 * 我们需要通过配置告诉它：
 *  - 哪些接口可以公开访问（登录、注册）
 *  - 哪些接口需要登录后才能访问
 *  - 哪些接口需要特定角色（如管理员）
 *  - 如何处理认证失败（返回 401）
 *  - 把我们的 JWT 过滤器加入到过滤器链中
 *
 * @Configuration 标记这是配置类
 * @EnableWebSecurity 启用 Spring Security 的 Web 安全支持
 */
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    /**
     * 注入 JWT 过滤器
     * 在过滤器链中，它会在 Spring Security 默认的用户名密码过滤器之前执行
     */
    private final JwtAuthenticationFilter jwtAuthenticationFilter;

    /**
     * 密码加密器 Bean
     *
     * BCryptPasswordEncoder 使用 BCrypt 算法加密密码：
     *  - 注册时：encode("123456") → "$2a$10$..."
     *  - 登录时：matches("123456", "$2a$10$...") → true
     *
     * 声明为 Bean 后，可以在 UserServiceImpl 中通过 @Autowired 注入
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    /**
     * 安全过滤器链配置 —— 最核心的配置方法
     *
     * SecurityFilterChain 定义了整个安全规则：
     *  - 哪些 URL 放行，哪些需要认证
     *  - Session 策略（JWT 是无状态的，不需要 Session）
     *  - 异常处理（401/403 的响应格式）
     *  - 添加自定义过滤器
     */
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                // 1. 禁用 CSRF（跨站请求伪造）保护
                //    原因：JWT 是无状态的，不使用 Cookie，不存在 CSRF 风险
                //    如果不禁用，POST 请求会被 Spring Security 拦截返回 403
                .csrf(AbstractHttpConfigurer::disable)

                // 2. 配置请求授权规则
                .authorizeHttpRequests(auth -> auth
                        // 登录和注册接口：允许所有人访问（无需 Token）
                        .requestMatchers("/api/auth/**").permitAll()

                        // 管理员接口：只有 ADMIN 角色才能访问
                        // hasRole("ADMIN") 会自动加 "ROLE_" 前缀，即检查 "ROLE_ADMIN"
                        .requestMatchers("/api/admin/**").hasRole("ADMIN")

                        // 其他所有请求：必须登录（携带有效 Token）才能访问
                        .anyRequest().authenticated()
                )

                // 3. Session 策略：STATELESS（无状态）
                //    Spring Security 默认使用 Session 维持登录状态
                //    JWT 模式下不需要 Session，每次请求都通过 Token 验证
                //    设置为无状态后，Spring Security 不会创建也不会使用 Session
                .sessionManagement(session ->
                        session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                )

                // 4. 异常处理
                .exceptionHandling(ex -> ex
                        // 401：未认证（没有 Token 或 Token 无效）
                        // 默认会重定向到登录页，我们改为返回 JSON
                        .authenticationEntryPoint((request, response, authException) -> {
                            response.setStatus(401);
                            response.setContentType("application/json;charset=UTF-8");
                            response.getWriter().write("{\"code\":401,\"message\":\"请先登录\",\"data\":null}");
                        })
                        // 403：已认证但权限不足（如普通用户访问管理员接口）
                        .accessDeniedHandler((request, response, accessDeniedException) -> {
                            response.setStatus(403);
                            response.setContentType("application/json;charset=UTF-8");
                            response.getWriter().write("{\"code\":403,\"message\":\"权限不足\",\"data\":null}");
                        })
                )

                // 5. 将 JWT 过滤器添加到过滤器链
                //    addFilterBefore(A, B) 表示：在过滤器 B 执行之前，先执行过滤器 A
                //    UsernamePasswordAuthenticationFilter 是 Spring Security 默认的表单登录过滤器
                //    我们的 JWT 过滤器要在它之前运行，这样认证信息就能提前存入上下文
                .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}

```


### 5. 登录接口

生成token并返回给前端

```java
public LoginResponse login(LoginRequest request) {  
    // 1. 根据用户名查询用户  
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();  
    queryWrapper.eq("username", request.getUsername());  
    User user = userMapper.selectOne(queryWrapper);  
  
    if (user == null) {  
        throw new RuntimeException("用户名或密码错误");  
    }  
  
    // 2. 验证密码  
    //    passwordEncoder.matches(明文密码, 数据库中的加密密码)  
    //    BCrypt 内部会提取盐值，重新计算哈希后比较  
    if (!passwordEncoder.matches(request.getPassword(), user.getPassword())) {  
        throw new RuntimeException("用户名或密码错误");  
    }  
  
    // 3. 生成 JWT Token    //    Token 中存储了用户 ID、用户名、角色，后续请求不需要再查数据库  
    String token = jwtUtil.generateToken(user.getId(), user.getUsername(), user.getRole());  
  
    // 4. 返回登录结果  
    return new LoginResponse(token, user.getUsername(), user.getNickname(), user.getRole());  
}
```