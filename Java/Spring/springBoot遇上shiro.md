---
title: SpringBoot邂逅Shiro
tags: SpringBoot,Shiro
grammar_cjkRuby: true
---
## 前言

本篇仅是记录集成的基础过程，至于shiro框架的基础概念和使用细节，可以自行查阅相关资料，本文不做讨论。

###  集成环境

部分jar包如下：
- springBoot: 1.5.8.RELEASE
- shiro-spring-boot-web-starter: 1.4.0-RC2
- shiro-redis: 3.1.0
- spring-boot-starter-data-redis

项目核心包为SpringBoot 1.5.8.RELEASE以及shiro-spring 1.4.0，预计集成redis，同时使用redis管理Session，所以追加了shiro-redis。关于数据库的jar包这里就不再赘述。

## 重新Session获取方式
Shiro默认从cookie获取SessionId以达到维持会话的目的。现在处理前后端分离，采用类似ajax请求的方式，通过在请求头中传递SessionId，因此需要重写Shiro获取SessionId的方式。

自定义MySessionManager类继承DefaultWebSessionManager类，可以重写getSessionId方法，代码如下 
```
import org.apache.commons.lang3.StringUtils;
import org.apache.shiro.web.servlet.ShiroHttpServletRequest;
import org.apache.shiro.web.session.mgt.DefaultWebSessionManager;
import org.apache.shiro.web.util.WebUtils;

import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import java.io.Serializable;

public class MySessionManager  extends DefaultWebSessionManager {

    private static final String AUTHORIZATION = "Authorization";

    private static final String REFERENCED_SESSION_ID_SOURCE = "Stateless request";

    public MySessionManager() {
        super();
    }

    @Override
    protected Serializable getSessionId(ServletRequest request, ServletResponse response) {
        String id = WebUtils.toHttp(request).getHeader(AUTHORIZATION);
        //如果请求头中有 Authorization 则其值为sessionId
        if (!StringUtils.isEmpty(id)) {
            request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID_SOURCE, REFERENCED_SESSION_ID_SOURCE);
            request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID, id);
            request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID_IS_VALID, Boolean.TRUE);
            return id;
        } else {
            //否则按默认规则从cookie取sessionId
            return super.getSessionId(request, response);
        }

    }
}
```
## 自定义授权Realm
ealm：域，Shiro从从Realm获取安全数据（如用户、角色、权限）：

- 就是说SecurityManager要验证用户身份，那么它需要从Realm获取相应的用户进行比较以确定用户身份是否合法；
- 也需要从Realm得到用户相应的角色/权限进行验证用户是否能进行操作；
- 可以把Realm看成DataSource，即安全数据源。
- 如我们之前的ini配置方式将使用org.apache.shiro.realm.text.IniRealm。

doGetAuthorizationInfo()方法用于控制用户权限获取。

doGetAuthenticationInfo()方法用于控制用户登录。

```
public class UserRealm extends AuthorizingRealm{
    //

    @Autowired
    private UserService userService;

    @Autowired
    private PermissionService permissionService;


    /**
     *  用于授权
     * @param principals
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        User user = (User)principals.getPrimaryPrincipal();
        if (user != null){
            SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
//            authorizationInfo.addRoles(permissionService.findPermissionRolesOfUser(userToken.getUser()));
            authorizationInfo.addStringPermissions(permissionService.findPermissionPrivilegesByUser());
            return authorizationInfo;
        }
        return null;
    }

    /**
     *  定义如何获取用户信息的业务逻辑，给shiro做登录
     * @param token
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {

        // 将AuthenticationToken强转为AuthenticationToken对象
        UsernamePasswordToken upToken = (UsernamePasswordToken) token;

        // 获取从表单传过来的用户名
        String username = upToken.getUsername();

        User user = userService.findByUsername(username);

        if (user == null){
            throw new UnknownAccountException("无此用户名！");
        }

        if (user.getIsDisable()){
            throw new LockedAccountException();
        }


        return new SimpleAuthenticationInfo(user, user.getPassword(), ByteSource.Util.bytes(user.getCredentialsSalt()), getName());
    }
}
```
## 配置使用

上面的类创建好如何让shiro检测到并使用？尽在在下面ShiroConfiguration类中：


```
@Configuration
public class ShiroConfiguration {

    @Autowired
    RedisProperties redisProperties;


    /**
     *  开启shiro aop注解支持.
     *  使用代理方式;所以需要开启代码支持;
     * @param securityManager
     * @return
     */
    @Bean("authorizationAttributeSourceAdvisor")
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
        //AuthorizationAttributeSourceAdvisor aasa = new AuthorizationAttributeSourceAdvisor();
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor  = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor .setSecurityManager(securityManager);
        return authorizationAttributeSourceAdvisor ;
    }


    @Bean
    public ShiroFilterFactoryBean shirFilter(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);


        Map<String, Filter> filters = new LinkedHashMap<String, Filter>();
        LogoutFilter logoutFilter = new LogoutFilter();
        logoutFilter.setRedirectUrl("/login");
        filters.put("logout", logoutFilter);
        shiroFilterFactoryBean.setFilters(filters);


        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
        //注意过滤器配置顺序 不能颠倒
        //配置退出 过滤器,其中的具体的退出代码Shiro已经替我们实现了，登出后跳转配置的loginUrl
        filterChainDefinitionMap.put("/logout", "logout");
        // 配置不会被拦截的链接 顺序判断
        filterChainDefinitionMap.put("/static/**", "anon");
        filterChainDefinitionMap.put("/api/login", "anon");
        filterChainDefinitionMap.put("/api/users/**", "anon");
        filterChainDefinitionMap.put("/login", "anon");
        filterChainDefinitionMap.put("/**", "authc");
        //配置shiro默认登录界面地址，前后端分离中登录界面跳转应由前端路由控制，后台仅返回json数据
        shiroFilterFactoryBean.setLoginUrl("/api/unauth");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;
    }

    /**
     * 凭证匹配器
     * @return
     */
    @Bean
    public HashedCredentialsMatcher hashedCredentialsMatcher(CacheManager cacheManager) {
        HashedCredentialsMatcher  hashedCredentialsMatcher = new HashedCredentialsMatcher (cacheManager);
        hashedCredentialsMatcher.setHashAlgorithmName(PasswordHelper.ALGORITHM);//散列算法:这里使用SHA-1算法;
        hashedCredentialsMatcher.setHashIterations(PasswordHelper.HASHITERATIONS);//散列的次数，比如散列两次，相当于 SHA-1(SHA-1(""));
        return hashedCredentialsMatcher;
    }

    /**
     *  自定义Realm，用于设置登录以及授权逻辑。
     *  spring允许用户通过depends-on属性指定bean前置依赖的bean,前置依赖的bean会在本bean实例化之前创建好
     * @param hashedCredentialsMatcher
     * @return
     */
    @Bean
    @DependsOn("lifecycleBeanPostProcessor")
    public Realm myShiroRealm(HashedCredentialsMatcher hashedCredentialsMatcher) {
        UserRealm myShiroRealm = new UserRealm();
        myShiroRealm.setCredentialsMatcher(hashedCredentialsMatcher);
        return myShiroRealm;
    }

    /**
     * 配置shiro redisManager
     * redisProperties会自动读取application.properties中关于redis的配置
     * @return
     */
    @Bean
    public RedisManager redisManager() {
        RedisManager redisManager = new RedisManager();
        redisManager.setHost(redisProperties.getHost());
        redisManager.setPort(redisProperties.getPort());
        redisManager.setTimeout(redisProperties.getTimeout());
		// 密码为可选项
        redisManager.setPassword(redisProperties.getPassword());
        return redisManager;
    }



    /**
     *  自定义sessionManager
     * @param redisSessionDAO
     * @return
     */
    @Bean
    public SessionManager sessionManager(RedisSessionDAO redisSessionDAO) {
        MySessionManager mySessionManager = new MySessionManager();
        mySessionManager.setSessionDAO(redisSessionDAO);
        return mySessionManager;
    }

    /**
     * RedisSessionDAO shiro sessionDao层的实现 通过redis
     * <p>
     * 使用的是shiro-redis开源插件
     */
    @Bean
    public RedisSessionDAO redisSessionDAO(RedisManager redisManager) {
        RedisSessionDAO redisSessionDAO = new RedisSessionDAO();
        redisSessionDAO.setRedisManager(redisManager);
        return redisSessionDAO;
    }


    /**
     * cacheManager 缓存 redis实现
     * @param redisManager
     * @return
     */
    @Bean("shiroRedisCacheManager")
    public CacheManager cacheManager(RedisManager redisManager) {
        RedisCacheManager redisCacheManager = new RedisCacheManager();
        redisCacheManager.setRedisManager(redisManager);
        return redisCacheManager;
    }


    /**
     * 这里使用shiro默认拦截器ShiroFilterChainDefinition
     *
     * @return
     */
    @Bean
    public ShiroFilterChainDefinition shiroFilterChainDefinition() {
        DefaultShiroFilterChainDefinition chainDefinition = new DefaultShiroFilterChainDefinition();
        return chainDefinition;
    }

    /**
     * shiro自动代理
     * DelegatingFilterProxy作用是自动到spring容器查找名字为shiroFilter（filter-name）的bean并把所有Filter的操作委托给它。
     * @return
     */
    @Bean
    @ConditionalOnMissingBean
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator daap = new DefaultAdvisorAutoProxyCreator();
        // shiro starter 默认实现未设置此属性，会导致开启事务的Service无法注入，因此替换默认设置
        daap.setProxyTargetClass(true);
        return daap;
    }

    /**
     * 注册全局异常处理
     * @return
     */
//    @Bean(name = "exceptionHandler")
//    public HandlerExceptionResolver handlerExceptionResolver() {
//        return new MyExceptionHandler();
//    }

}
```
redisProperties和securityManager在idea中可能会报找不到been,该错误可以忽略，程序会自动寻找默认注入。

这里没有实现自定义的全局异常控制，有需要时可以考虑使用SpringMVC的全局异常捕获来处理异常。

若想redisProperties能获取默认配置，需要先在application.properties中对redis进行配置：
```
#redis
spring.redis.host=个人地址
spring.redis.password=若无密码可注释并不配置此项
spring.redis.port=6380
spring.redis.database=8
spring.redis.timeout=60000
```

凭证匹配器部分使用了PasswordHelper类为自己方便添加用户时两者能够统一而写的工具类：
```
public class PasswordHelper {

    public static final String ALGORITHM = "SHA-1";

    public static final int HASHITERATIONS = 2;

    private static final int SALT_SIZE = 22;


    public static String generateSalt(){
        byte[] salt = SecurityUtils.generateSalt(SALT_SIZE);
        return SecurityUtils.encodeHex(salt);
    }

    public static String encryptPassword(User user) {
        String newPassword = new SimpleHash(
                ALGORITHM,
                user.getPassword(),
                ByteSource.Util.bytes(user.getCredentialsSalt()),
                HASHITERATIONS).toHex();
        return  newPassword;
    }

}
```
里面获取盐值的时SecurityUtils类涉及到的两个方法如下，本质是通过SecureRandom获取的随机数：
```
    private static SecureRandom random = new SecureRandom();

    /**
     * 生成指定为数的随机的Byte[]作为salt.
     * @param numBytes
     * @return
     */
    public static byte[] generateSalt(int numBytes) {
        Validate.isTrue(numBytes > 0, "numBytes argument must be a positive integer (1 or larger)", numBytes);

        byte[] bytes = new byte[numBytes];
        random.nextBytes(bytes);
        return bytes;
    }


    public static String encodeHex(byte[] input) {
        return new String(Hex.encodeHex(input));
    }
```
## 登录api
前后端分离中通过api返回登录状态，前端通过该状态决定是否成功，同时成功会返回token，后续授权均需通过该token鉴权：
```
@RestController
@RequestMapping("api/")
public class LoginApiController {

    @PostMapping("login")
    public ReturnResult adminLogin(User user){
        ReturnResult result = new ReturnResult();
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken(user.getUsername(), user.getPassword());
        try {
            subject.login(token);
            result.setToken(subject.getSession().getId());
            result.setMsg("登录成功");
            result.setCode(200);
        }  catch (IncorrectCredentialsException e) {
            result.setMsg("密码错误");
            result.setCode(400);
        } catch (LockedAccountException e) {
            result.setMsg("登录失败，该用户已被冻结");
            result.setCode(400);
        } catch (AuthenticationException e) {
            result.setMsg("该用户不存在");
            result.setCode(400);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * 未登录，shiro应重定向到登录界面，此处返回未登录状态信息由前端控制跳转页面
     * @return
     */
    @RequestMapping(value = "/unauth")
    public ReturnResult unauth() {
        ReturnResult result = new ReturnResult();
        result.setMsg("未登录");
        result.setCode(400);
        return result;
    }

}
```
里面的返回值格式可以自行创建，这里为个人使用的ReturnResult，分享如下：
```
public class ReturnResult implements Serializable {
    private int code = 0;
    private String msg = null;
    private Object token;
    private Object result;

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public Object getResult() {
        return result;
    }

    public void setResult(Object result) {
        this.result = result;
    }

    public Object getToken() {
        return token;
    }

    public void setToken(Object token) {
        this.token = token;
    }

    public String toJsonString(){

        JSONObject json = new JSONObject(this);
        return json.toString();
    }
}
```

## 参考资料
[在前后端分离的SpringBoot项目中集成Shiro权限框架](https://blog.csdn.net/u013615903/article/details/78781166/)
[跟我学Shiro目录贴](http://jinnianshilongnian.iteye.com/blog/2018398)