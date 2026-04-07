# 单元测试编写

单元测试如何编写
<!--more-->

场景： 在实习中，我发现在团队合作时，需要单独完成某个流程中的一环，当你的上游没有完成代码时，你需要写单元测试，来测试你的代码，这时你需要mock数据。

## 1 基本概念- what

什么是单元测试： 单独测试你的某一个类/方法是否正确，不依赖数据库，上下游，只关心逻辑
<br/>
什么是mock： 造一个假对象，模拟真实对象行为
比如：当你调用一个外部函数 get session 时，你直接返回你定义好的假的 session 对象
<br/>

## 2 实战 - how

### 2.1 第一步：理解你要测什么

要写测试前，先给出这些：
- 这个类有几个 public 方法？
- 每个方法的正常流程是什么？
- 每个方法里有哪些地方可能会抛异常（即失败场景）？

```java
@Service  
public class UserServiceImpl implements UserService {  
  
    @Resource  
    private UserMapper userMapper;  
  
  
    @Override  
    public UserInfoVO getLoginUser(HttpServletRequest req) {  
        //1. 校验用户是否登录，如果未登录则抛出异常  
        SessionUser sessionUser = (SessionUser) req.getSession().getAttribute(UserConstant.USER);  
        ThrowUtils.throwIf(sessionUser == null, ErrorCode.NOT_LOGIN);  
  
        //2. 校验用户是否存在  
        User checkUser = userMapper.selectByPrimaryKey(sessionUser.getId());  
        ThrowUtils.throwIf(checkUser == null, ErrorCode.USER_NOT_EXIST);  
  
        //3. 校验用户是否被禁用  
        ThrowUtils.throwIf(checkUser.getUserStatus().equals(UserStatusEnum.DISABLED.getCode()), ErrorCode.USER_DISABLE);  
  
        //4. 返回用户信息  
        return UserInfoVO.toUserInfoVO(checkUser);  
    }
}
```

比如这是一个获取登录用户的方法， 现在要测试这个方法
给出上面的三个问题的答案：
```text
1. 一个public 方法
   
2. getLoginUser：正常流程：校验session是否登录，校验用户是否存在，校验用户是否被禁用，返回用户信息      
可能抛出异常：session中没有用户，用户在登录时被删除了，用户在登陆时被禁用了
被删除了，用户在登陆时被禁用了，新密码与确认密码不一致，旧密码不正确，操作数据库失败
```

### 2.2 第二步：列出测试清单
根据上面的正常流程和异常，列出测试清单
```text
getLoginUser_success
getLoginUser_未登录
getLoginUser_处于登录态被删除
getLoginUser_处于登录态被禁用
```

### 2.3 第三步： 编写代码，搭建测试骨架


- @@ExtendWith(MockitoExtension.class)   启用mockito
- 对于要测试的类使用 @InjectMocks，表示将@Mock造的假对象注入
- 对于要模拟的假类 使用时@Mock  
- 对于测试方法，使用@Test

```java
  
@ExtendWith(MockitoExtension.class)  
public class UserServiceImplTest {  
  
    @Mock  
    private UserMapper userMapper;  
  
    @Mock  
    private HttpSession httpSession;  
  
    @Mock  
    private HttpServletRequest httpServletRequest;  
  
    @InjectMocks  
    private UserServiceImpl userService;  
  
    @Test  
    @DisplayName("获取用户信息成功")  
    void getLoginUser_success() {  
        // given  
        // when  
        // then  
    }  
  
    @Test  
    @DisplayName("获取用户信息失败 - 未登录")  
    void getLoginUser_未登录() {  
        // given  
        // when  
        // then        
        });  
    }  
}
```

### 2.4 第四步：编写测试流程
getLoginUser 他的执行路径是：
```text
req.getSession() → 拿到 session
session.getAttribute(...) → 拿到 SessionUser
userMapper.selectByPrimaryKey(id) → 拿到 User
检查 user 状态 != DISABLED
返回 UserInfoVO
```

你的测试要做的就是：模拟这条路径上每一步的外部依赖返回值，让代码能走通。

可以看到上面测试方法里的三个流程： given，when，then


given（准备阶段）：  
- 你需要问自己：这个方法依赖了哪些外部调用？每个调用应该返回什么？(可以看这个类的字段，方法参数等来判断有哪些)
```
httpServletRequest.getSession() → 应该返回什么？
httpSession.getAttribute(UserConstant.USER) → 应该返回什么？
userMapper.selectByPrimaryKey(...) → 应该返回什么？
```
这三个就是你需要用 when(...).thenReturn(...) 来 mock 的。****


when（执行阶段）
就一行——调用被测方法。

then（验证阶段）
拿到返回的 UserInfoVO，用 assertNotNull 和 assertEquals 验证里面的字段是否符合预期。


这是方法成功的测试
```java
@Test  
@DisplayName("获取用户信息成功")  
void getLoginUser_success() {  
    // given  
  
  
  //创建模拟对象
    SessionUser sessionUser = new SessionUser();  
    sessionUser.setId(1L);  
    sessionUser.setUserName("cheems");  
    sessionUser.setUserRole("admin");  
  
    User user = new User();  
    user.setId(1L);  
    user.setUserName("cheems");  
    user.setUserAccount("user123");  
    user.setUserPassword("123456a@");  
    user.setUserEmail("a@a.com");  
    user.setUserRole("admin");  
    user.setUserStatus(0);  
    user.setCreateTime(LocalDateTime.now());  
    user.setUpdateTime(LocalDateTime.now());  
    user.setIsDelete(0);  
  
  //设定假的返回
    when(httpServletRequest.getSession()).thenReturn(httpSession);  
when(httpSession.getAttribute(UserConstant.USER)).thenReturn(sessionUser);  
    when(userMapper.selectByPrimaryKey(1L)).thenReturn(user);  
  
  
    // when  
    UserInfoVO loginUser = userService.getLoginUser(httpServletRequest);  
  
    // then  
    assertNotNull(loginUser);  
    assertEquals("cheems", loginUser.getUserName());  
    assertEquals("user123", loginUser.getUserAccount());  
    assertEquals("a@a.com", loginUser.getUserEmail());  
    assertEquals("admin", loginUser.getUserRole());  
    assertEquals(0, loginUser.getUserStatus());  
  
}
```


这是异常的测试：
<br/>
异常测试：在某一步制造异常条件 → 验证抛出了预期的异常
<br/>
getAttribute 返回 null 时会抛异常
所以你的 mock 只需要让 getAttribute 返回 null（或者干脆不 mock 它，因为 mock 对象默认返回 null）
后面的 userMapper 根本不会被调用到，所以不需要 mock 它

```java
@Test  
@DisplayName("获取用户信息失败 - 未登录")  
void getLoginUser_未登录() {  
    // given  
    when(httpServletRequest.getSession()).thenReturn(httpSession);  
    when(httpSession.getAttribute(UserConstant.USER)).thenReturn(null);  
  
    // when  
    // then    assertThrows(BusinessException.class,()->{  
        userService.getLoginUser(httpServletRequest);  
    });  
}
```



---

> 作者: cheems  
> URL: http://localhost:59658/java/%E6%B5%8B%E8%AF%95/5c385d4c/  

