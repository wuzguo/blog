---
title: Shiro学习笔记
date: 2016-08-02 02:43:12 
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wzguo
authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」
categories: 后台
tags: 
	- shiro
keywords: shiro
photos:
	- /blog/images/201608/3.png
description: Apache Shiro权限管理框架的使用
---

1.Shiro介绍
Apache Shiro是一个强大易用的Java安全框架，提供了认证、授权、加密和会话管理等功能：
- 认证 用户身份识别，常被称为用户“登录”
- 授权 访问控制
- 密码加密 保护或隐藏数据防止被偷窥
- 会话管理 每用户相关的时间敏感的状态

对于任何一个应用程序，Shiro都可以提供全面的安全管理服务。并且相对于其他安全框架，Shiro要简单的多。

2.Shiro架构
首先，来了解一下Shiro的三个核心组件：Subject, SecurityManager 和 Realms. 如下图：
![](/blog/images/201608/1.png)

Subject：即“当前操作用户”。但是，在Shiro中，Subject这一概念并不仅仅指人，也可以是第三方进程、后台帐户（Daemon Account）或其他类似事物。它仅仅意味着“当前跟软件交互的东西”。但考虑到大多数目的和用途，你可以把它认为是Shiro的“用户”概念。
Subject代表了当前用户的安全操作，SecurityManager则管理所有用户的安全操作。

SecurityManager：它是Shiro框架的核心，典型的Facade模式，Shiro通过SecurityManager来管理内部组件实例，并通过它来提供安全管理的各种服务。
SecurityManager默认实现结构：
![](/blog/images/201608/5.png)

Realm：域，Shiro从Realm获取安全数据（如用户、角色、权限），就是说SecurityManager要验证用户身份，那么它需要从Realm获取相应的用户进行比较以确定用户身份是否合法；也需要从Realm得到用户相应的角色/权限进行验证用户是否能进行操作；可以把Realm看成DataSource，即安全数据源。
从这个意义上讲，Realm实质上是一个安全相关的DAO：它封装了数据源的连接细节，并在需要时将相关数据提供给Shiro。当配置Shiro时，你必须至少指定一个Realm，用于认证和（或）授权。配置多个Realm是可以的，但是至少需要一个。

Shiro内置了可以连接大量安全数据源（又名目录）的Realm，如LDAP、关系数据库（JDBC）、类似INI的文本配置资源以及属性文件等。如果缺省的Realm不能满足需求，你还可以插入代表自定义数据源的自己的Realm实现。

Shiro缺省Realm实现：
![](/blog/images/201608/4.png)


实现自己的Realm
```java
......
public class UserRealm extends AuthorizingRealm {

    private static final Logger logger = LoggerFactory.getLogger(UserRealm.class);

    @Resource
    private IUserService userService;

    /**
     * 清楚授权的缓存，授权缓存以用户对象为键
     *
     * @param principals
     */
    @Override
    protected void clearCachedAuthorizationInfo(PrincipalCollection principals) {
        logger.debug("[UserRealm] clearCachedAuthorizationInfo begin");
        Cache cache = this.getAuthorizationCache();
        Set<Object> keys = cache.keys();
        for (Object object : keys) {
            logger.debug("[UserRealm] clearCachedAuthorizationInfo object: " + object);
        }
        super.clearCachedAuthorizationInfo(principals);
        logger.debug("[UserRealm] clearCachedAuthorizationInfo end");
    }

    /**
     * 授权，如果不使用缓存，每次访问新页面都会执行这个方法
     *
     * @param principals
     * @return
     */
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        User user = ((User) principals.getPrimaryPrincipal());
        String code = user.getCode();
        System.out.println(user.getId() + "," + user.getNickname());
        List<Role> roles = userService.listRoleByUserCode(code);
        List<AuthResource> authResources = userService.listResByUserCode(code);
        List<String> permissions = new ArrayList<String>();
        for (AuthResource resource : authResources) {
            permissions.add(resource.getUrl());
        }
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        // info.setRoles(new HashSet<String>(roles));
        info.setStringPermissions(new HashSet<String>(permissions));
        return info;
    }

    /**
     * 清除认证的缓存
     *
     * @param principals
     */
    @Override
    protected void clearCachedAuthenticationInfo(PrincipalCollection principals) {
        logger.debug("[UserRealm] clearCachedAuthenticationInfo begin");

        Cache cache = this.getAuthenticationCache();
        Set<Object> keys = cache.keys();
        for (Object object : keys) {
            logger.debug("[UserRealm] clearCachedAuthorizationInfo object: " + object);
        }

        // 认证的缓存以用户名为键，需要手动处理，否则退出登录时登录信息将不能被清除
        User user = ((User) principals.getPrimaryPrincipal());
        SimplePrincipalCollection simplePrincipalCollection = new SimplePrincipalCollection(user.getUsername(), this.getName());
        super.clearCachedAuthenticationInfo(simplePrincipalCollection);
        logger.debug("[UserRealm] clearCachedAuthenticationInfo end");
    }

    /**
     * 获取认证信息，登录的时候调用
     *
     * @param token
     * @return
     * @throws AuthenticationException
     */
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        String username = token.getPrincipal().toString();
        String password = new String((char[]) token.getCredentials());
        // 获取用户信息
        User user = userService.doGetUserInfo(username, password);
        if (user == null) {
            logger.debug("[UserRealm] doGetAuthenticationInfo user is null");
            throw new AuthenticationException("username invalid");
        }
        SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(user, user.getSerct(), this.getName());
        // 设置Salt值，增强密码破解的难度
        info.setCredentialsSalt(ByteSource.Util.bytes(user.getUsername()));
        return info;
    }
}

......
```
Shiro完整架构图：

![](/blog/images/201608/2.png)

3.Shiro认证与授权
认证就是验证用户身份的过程。在认证过程中，用户需要提交实体信息(Principals)和凭据信息(Credentials)以检验用户是否合法。最常见的“实体/凭证”组合便是“用户名/密码”组合。被 Shiro 保护的资源，才会经过认证与授权过程。使用 Shiro 对 URL 进行保护可以参见“与 Spring 集成”章节。用户访问受 Shiro 保护的 URL；例如 http://host/security/action.do，Shiro 首先检查用户是否已经通过认证，如果未通过认证检查，则跳转到登录页面，否则进行授权检查。认证过程需要通过 Realm 来获取用户及密码信息，通常情况我们实现 JDBC Realm，此时用户认证所需要的信息从数据库获取。如果使用了缓存，除第一次外用户信息从缓存获取。认证通过后接受 Shiro 授权检查，授权检查同样需要通过 Realm 获取用户权限信息。Shiro 需要的用户权限信息包括 Role 或 Permission，可以是其中任何一种或同时两者，具体取决于受保护资源的配置。如果用户权限信息未包含 Shiro 需要的 Role 或 Permission，授权不通过。只有授权通过，才可以访问受保护 URL 对应的资源，否则跳转到“未经授权页面”。

3.1 身份认证流程
1.首先创建一个SecurityManager工厂
2.接着获取SecurityManager并绑定到SecurityUtils，这是一个全局设置，设置一次即可；
3.通过SecurityUtils得到Subject，其会自动绑定到当前线程；如果在web环境在请求结束时需要解除绑定；然后获取身份验证的Token，如用户名/密码；
4.调用subject.login方法进行登录，其会自动委托给SecurityManager.login方法进行登录；
5.如果身份验证失败请捕获AuthenticationException或其子类，常见的如： DisabledAccountException（禁用的帐号）、LockedAccountException（锁定的帐号）、UnknownAccountException（错误的帐号）、ExcessiveAttemptsException（登录失败次数过多）、IncorrectCredentialsException （错误的凭证）、ExpiredCredentialsException（过期的凭证）等，具体请查看其继承关系；对于页面的错误消息展示，最好使用如“用户名/密码错误”而不是“用户名错误”/“密码错误”，防止一些恶意用户非法扫描帐号库；
6.最后可以调用subject.logout退出，其会自动委托给SecurityManager.logout方法退出。
	 
从如上代码可总结出身份验证的步骤：
1.收集用户身份/凭证，即如用户名/密码；
2.调用Subject.login进行登录，如果失败将得到相应的AuthenticationException异常，根据异常提示用户错误信息；否则登录成功；
3.最后调用Subject.logout进行退出操作。
 
流程图如下：
![](/blog/images/201608/3.png)

实现代码如下：
```java
@Controller
@RequestMapping("/")
public class LoginController {

    @RequestMapping(value = "/login", method = RequestMethod.GET)
    public String login() {
        return "login";
    }

    @RequestMapping(value = "/login", method = RequestMethod.POST)
    public String login(String username, String password, Model model) {
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken(username, password);
        try {
            subject.login(token);
        } catch (AuthenticationException e) {
            e.printStackTrace();
        }
    }
}
```
3.2 Authenticator及AuthenticationStrategy
Authenticator的职责是验证用户帐号，是Shiro API中身份验证核心的入口点： 
```java
public AuthenticationInfo authenticate(AuthenticationToken authenticationToken)  throws AuthenticationException;  
```
如果验证成功，将返回AuthenticationInfo验证信息；此信息中包含了身份及凭证；如果验证失败将抛出相应的AuthenticationException实现。
 
SecurityManager接口继承了Authenticator，另外还有一个ModularRealmAuthenticator实现，其委托给多个Realm进行验证，验证规则通过AuthenticationStrategy接口指定，默认提供的实现：

FirstSuccessfulStrategy：只要有一个Realm验证成功即可，只返回第一个Realm身份验证成功的认证信息，其他的忽略；
AtLeastOneSuccessfulStrategy：只要有一个Realm验证成功即可，和FirstSuccessfulStrategy不同，返回所有Realm身份验证成功的认证信息；
AllSuccessfulStrategy：所有Realm验证成功才算成功，且返回所有Realm身份验证成功的认证信息，如果有一个失败就失败了。
 
ModularRealmAuthenticator默认使用AtLeastOneSuccessfulStrategy策略。

3.3 授权流程
授权，也叫访问控制，即在应用中控制谁能访问哪些资源（如访问页面/编辑数据/页面操作等）。在授权中需了解的几个关键对象：主体（Subject）、资源（Resource）、权限（Permission）、角色（Role）。

主体，即访问应用的用户，在Shiro中使用Subject代表该用户。用户只有授权后才允许访问相应的资源。
资源，在应用中用户可以访问的任何东西，比如访问JSP页面、查看/编辑某些数据、访问某个业务方法、打印文本等等都是资源。用户只要授权后才能访问。
权限，安全策略中的原子授权单位，通过权限我们可以表示在应用中用户有没有操作某个资源的权力。即权限表示在应用中用户能不能访问某个资源，如：访问用户列表页面，查看/新增/修改/删除用户数据（即很多时候都是CRUD（增查改删）式权限控制），打印文档等等...
 
如上可以看出，权限代表了用户有没有操作某个资源的权利，即反映在某个资源上的操作允不允许，不反映谁去执行这个操作。所以后续还需要把权限赋予给用户，即定义哪个用户允许在某个资源上做什么操作（权限），Shiro不会去做这件事情，而是由实现人员提供。
 
角色，角色代表了操作集合，可以理解为权限的集合，一般情况下我们会赋予用户角色而不是权限，即这样用户可以拥有一组权限，赋予权限时比较方便。典型的如：项目经理、技术总监、CTO、开发工程师等都是角色，不同的角色拥有一组不同的权限。
隐式角色：即直接通过角色来验证用户有没有操作权限，如在应用中CTO、技术总监、开发工程师可以使用打印机，假设某天不允许开发工程师使用打印机，此时需要从应用中删除相应代码；再如在应用中CTO、技术总监可以查看用户、查看权限；突然有一天不允许技术总监查看用户、查看权限了，需要在相关代码中把技术总监角色从判断逻辑中删除掉；即粒度是以角色为单位进行访问控制的，粒度较粗；如果进行修改可能造成多处代码修改。
显示角色：在程序中通过权限控制谁能访问某个资源，角色聚合一组权限集合；这样假设哪个角色不能访问某个资源，只需要从角色代表的权限集合中移除即可；无须修改多处代码；即粒度是以资源/实例为单位的；粒度较细。

流程如下：
1.首先调用Subject.isPermitted*/hasRole*接口，其会委托给SecurityManager，而SecurityManager接着会委托给Authorizer；
2.Authorizer是真正的授权者，如果我们调用如isPermitted(“user:view”)，其首先会通过PermissionResolver把字符串转换成相应的Permission实例；
3.在进行授权之前，其会调用相应的Realm获取Subject相应的角色/权限用于匹配传入的角色/权限；
4.Authorizer会判断Realm的角色/权限是否和传入的匹配，如果有多个Realm，会委托给ModularRealmAuthorizer进行循环判断，如果匹配如isPermitted*/hasRole*会返回true，否则返回false表示授权失败。
 
ModularRealmAuthorizer进行多Realm匹配流程：
1.首先检查相应的Realm是否实现了实现了Authorizer；
2.如果实现了Authorizer，那么接着调用其相应的isPermitted*/hasRole*接口进行匹配；
3.如果有一个Realm匹配那么将返回true，否则返回false。
 
如果Realm进行授权的话，应该继承AuthorizingRealm，其流程是：
1.如果调用hasRole*，则直接获取AuthorizationInfo.getRoles()与传入的角色比较即可；
2.首先如果调用如isPermitted(“user:view”)，首先通过PermissionResolver将权限字符串转换成相应的Permission实例，默认使用WildcardPermissionResolver，即转换为通配符的WildcardPermission；
3.通过AuthorizationInfo.getObjectPermissions()得到Permission实例集合；通过AuthorizationInfo. getStringPermissions()得到字符串集合并通过PermissionResolver解析为Permission实例；然后获取用户的角色，并通过RolePermissionResolver解析角色对应的权限集合（默认没有实现，可以自己提供）；
4.接着调用Permission. implies(Permission p)逐个与传入的权限比较，如果有匹配的则返回true，否则false。 
 
3.4 Authorizer、PermissionResolver及RolePermissionResolver
Authorizer的职责是进行授权（访问控制），是Shiro API中授权核心的入口点，其提供了相应的角色/权限判断接口，具体请参考其Javadoc。SecurityManager继承了Authorizer接口，且提供了ModularRealmAuthorizer用于多Realm时的授权匹配。PermissionResolver用于解析权限字符串到Permission实例，而RolePermissionResolver用于根据角色解析相应的权限集合。
 
我们可以通过如下配置更改Authorizer实现：
```xml
......
<!--自定义permissionResolver-->
<bean id="urlPermissionResolver" class="com.github.wzguo.service.shiro.resolver.WebUrlPermissionResolver"/>

<bean id="authorizer" class="org.apache.shiro.authz.ModularRealmAuthorizer">
	<property name="permissionResolver" ref="urlPermissionResolver"/>
</bean>

<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
	<property name="authorizer" ref="authorizer"/>
</bean>
......
```
流程图如下：
![](/blog/images/201608/6.png)


实现代码如下：
```java
@Controller
@RequestMapping(value="user")
public class UserController {
	@RequestMapping(params = "myjsp")
	public String home() {
		Subject currentUser = SecurityUtils.getSubject();
		if(currentUser.isPermitted("user.do?myjsp")){
			return "my";
		}else{
			return "error/noperms";
		}
	}
	@RequestMapping(params = "notmyjsp")
	public String nopermission() {
		Subject currentUser = SecurityUtils.getSubject();
		if(currentUser.isPermitted("user.do?notmyjsp")){
			return "notmyjsp";
		}else{
			return "error/noperms";
		}
	}
}
```
4.拦截器
4.1 拦截器介绍
Shiro使用了与Servlet一样的Filter接口进行扩展,首先下图是Shiro拦截器的基础类图：
![](/blog/images/201608/7.png)

NameableFilter
NameableFilter给Filter起个名字，如果没有设置默认就是FilterName；还记得之前的如authc吗？当我们组装拦截器链时会根据这个名字找到相应的拦截器实例；
 
OncePerRequestFilter
OncePerRequestFilter用于防止多次执行Filter的；也就是说一次请求只会走一次拦截器链；另外提供enabled属性，表示是否开启该拦截器实例，默认enabled=true表示开启，如果不想让某个拦截器工作，可以设置为false即可。
 
ShiroFilter
ShiroFilter是整个Shiro的入口点，用于拦截需要安全控制的请求进行处理，这个之前已经用过了。
 
AdviceFilter
AdviceFilter提供了AOP风格的支持，类似于SpringMVC中的Interceptor：
```java
boolean preHandle(ServletRequest request, ServletResponse response) throws Exception  
void postHandle(ServletRequest request, ServletResponse response) throws Exception  
void afterCompletion(ServletRequest request, ServletResponse response, Exception exception) throws Exception;  
```
preHandler：类似于AOP中的前置增强；在拦截器链执行之前执行；如果返回true则继续拦截器链；否则中断后续的拦截器链的执行直接返回；进行预处理（如基于表单的身份验证、授权）
postHandle：类似于AOP中的后置返回增强；在拦截器链执行完成后执行；进行后处理（如记录执行时间之类的）；
afterCompletion：类似于AOP中的后置最终增强；即不管有没有异常都会执行；可以进行清理资源（如接触Subject与线程的绑定之类的）；
 
PathMatchingFilter
PathMatchingFilter提供了基于Ant风格的请求路径匹配功能及拦截器参数解析的功能，如“roles[admin,user]”自动根据“，”分割解析到一个路径参数配置并绑定到相应的路径：
```java
boolean pathsMatch(String path, ServletRequest request)  
boolean onPreHandle(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception
```
pathsMatch：该方法用于path与请求路径进行匹配的方法；如果匹配返回true；
onPreHandle：在preHandle中，当pathsMatch匹配一个路径后，会调用opPreHandler方法并将路径绑定参数配置传给mappedValue；然后可以在这个方法中进行一些验证（如角色授权），如果验证失败可以返回false中断流程；默认返回true；也就是说子类可以只实现onPreHandle即可，无须实现preHandle。如果没有path与请求路径匹配，默认是通过的（即preHandle返回true）。
 
AccessControlFilter
AccessControlFilter提供了访问控制的基础功能；比如是否允许访问/当访问拒绝时如何处理等：
```java
abstract boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception;  
boolean onAccessDenied(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception;  
abstract boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception;   
```
isAccessAllowed：表示是否允许访问；mappedValue就是[urls]配置中拦截器参数部分，如果允许访问返回true，否则false；
onAccessDenied：表示当访问拒绝时是否已经处理了；如果返回true表示需要继续处理；如果返回false表示该拦截器实例已经处理了，将直接返回即可。

onPreHandle会自动调用这两个方法决定是否继续处理：
```java
boolean onPreHandle(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {  
    return isAccessAllowed(request, response, mappedValue) || onAccessDenied(request, response, mappedValue);  
}  
```
另外AccessControlFilter还提供了如下方法用于处理如登录成功后/重定向到上一个请求： 
```java
void setLoginUrl(String loginUrl) //身份验证时使用，默认/login.jsp  
String getLoginUrl()  
Subject getSubject(ServletRequest request, ServletResponse response) //获取Subject实例  
boolean isLoginRequest(ServletRequest request, ServletResponse response)//当前请求是否是登录请求  
void saveRequestAndRedirectToLogin(ServletRequest request, ServletResponse response) throws IOException //将当前请求保存起来并重定向到登录页面  
void saveRequest(ServletRequest request) //将请求保存起来，如登录成功后再重定向回该请求  
void redirectToLogin(ServletRequest request, ServletResponse response) //重定向到登录页面
```
比如基于表单的身份验证就需要使用这些功能。
 
到此基本的拦截器就完事了，如果我们想进行访问访问的控制就可以继承AccessControlFilter；如果我们要添加一些通用数据我们可以直接继承PathMatchingFilter。
 
4.2 拦截器链
Shiro对Servlet容器的FilterChain进行了代理，即ShiroFilter在继续Servlet容器的Filter链的执行之前，通过ProxiedFilterChain对Servlet容器的FilterChain进行了代理；即先走Shiro自己的Filter体系，然后才会委托给Servlet容器的FilterChain进行Servlet容器级别的Filter链执行；Shiro的ProxiedFilterChain执行流程：1、先执行Shiro自己的Filter链；2、再执行Servlet容器的Filter链（即原始的Filter）。
而ProxiedFilterChain是通过FilterChainResolver根据配置文件中[urls]部分是否与请求的URL是否匹配解析得到的。 
```java
FilterChain getChain(ServletRequest request, ServletResponse response, FilterChain originalChain);
```
即传入原始的chain得到一个代理的chain。
Shiro内部提供了一个路径匹配的FilterChainResolver实现：PathMatchingFilterChainResolver，其根据[urls]中配置的url模式（默认Ant风格）=拦截器链和请求的url是否匹配来解析得到配置的拦截器链的；而PathMatchingFilterChainResolver内部通过FilterChainManager维护着拦截器链，比如DefaultFilterChainManager实现维护着url模式与拦截器链的关系。因此我们可以通过FilterChainManager进行动态动态增加url模式与拦截器链的关系。
 
DefaultFilterChainManager会默认添加org.apache.shiro.web.filter.mgt.DefaultFilter中声明的拦截器：
```java
public enum DefaultFilter {  
    anon(AnonymousFilter.class),  
    authc(FormAuthenticationFilter.class),  
    authcBasic(BasicHttpAuthenticationFilter.class),  
    logout(LogoutFilter.class),  
    noSessionCreation(NoSessionCreationFilter.class),  
    perms(PermissionsAuthorizationFilter.class),  
    port(PortFilter.class),  
    rest(HttpMethodPermissionFilter.class),  
    roles(RolesAuthorizationFilter.class),  
    ssl(SslFilter.class),  
    user(UserFilter.class);  
}   
```
如果要注册自定义拦截器，IniSecurityManagerFactory/WebIniSecurityManagerFactory在启动时会自动扫描ini配置文件中的[filters]/[main]部分并注册这些拦截器到DefaultFilterChainManager；且创建相应的url模式与其拦截器关系链。如果使用Spring后续章节会介绍如果注册自定义拦截器。
 
如果想自定义FilterChainResolver，可以通过实现WebEnvironment接口完成：
```java
public class MyIniWebEnvironment extends IniWebEnvironment {  
    @Override  
    protected FilterChainResolver createFilterChainResolver() {  
        //在此处扩展自己的FilterChainResolver  
        return super.createFilterChainResolver();  
    }  
}
```
FilterChain之间的关系。如果想动态实现url-拦截器的注册，就可以通过实现此处的FilterChainResolver来完成，比如：
```java
//1、创建FilterChainResolver  
PathMatchingFilterChainResolver filterChainResolver = new PathMatchingFilterChainResolver();  
//2、创建FilterChainManager  
DefaultFilterChainManager filterChainManager = new DefaultFilterChainManager();  
//3、注册Filter  
for(DefaultFilter filter : DefaultFilter.values()) {  
    filterChainManager.addFilter(  
        filter.name(), (Filter) ClassUtils.newInstance(filter.getFilterClass()));  
}  
//4、注册URL-Filter的映射关系  
filterChainManager.addToChain("/login.jsp", "authc");  
filterChainManager.addToChain("/unauthorized.jsp", "anon");  
filterChainManager.addToChain("/**", "authc");  
filterChainManager.addToChain("/**", "roles", "admin");  
  
//5、设置Filter的属性  
FormAuthenticationFilter authcFilter =  
         (FormAuthenticationFilter)filterChainManager.getFilter("authc");  
authcFilter.setLoginUrl("/login.jsp");  
RolesAuthorizationFilter rolesFilter =  
          (RolesAuthorizationFilter)filterChainManager.getFilter("roles");  
rolesFilter.setUnauthorizedUrl("/unauthorized.jsp");  
  
filterChainResolver.setFilterChainManager(filterChainManager);  
return filterChainResolver;   
```
此处自己去实现注册filter，及url模式与filter之间的映射关系。可以通过定制FilterChainResolver或FilterChainManager来完成诸如动态URL匹配的实现。
 
然后再web.xml中进行如下配置Environment：  
```xml
<context-param>  
	<param-name>shiroEnvironmentClass</param-name> 
	<param-value>com.github.zhangkaitao.shiro.chapter8.web.env.MyIniWebEnvironment</param-value>  
</context-param>   
```
4.3 自定义拦截器
通过自定义自己的拦截器可以扩展一些功能，诸如动态url-角色/权限访问控制的实现、根据Subject身份信息获取用户信息绑定到Request（即设置通用数据）、验证码验证、在线用户信息的保存等等，因为其本质就是一个Filter；所以Filter能做的它就能做。
 
基于表单登录拦截器 
```java
......
public class FormLoginFilter extends PathMatchingFilter {  
    private String loginUrl = "/login.jsp";  
    private String successUrl = "/";  
    @Override  
    protected boolean onPreHandle(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {  
        if(SecurityUtils.getSubject().isAuthenticated()) {  
            return true;//已经登录过  
        }  
        HttpServletRequest req = (HttpServletRequest) request;  
        HttpServletResponse resp = (HttpServletResponse) response;  
        if(isLoginRequest(req)) {  
            if("post".equalsIgnoreCase(req.getMethod())) {//form表单提交  
                boolean loginSuccess = login(req); //登录  
                if(loginSuccess) {  
                    return redirectToSuccessUrl(req, resp);  
                }  
            }  
            return true;//继续过滤器链  
        } else {//保存当前地址并重定向到登录界面  
            saveRequestAndRedirectToLogin(req, resp);  
            return false;  
        }  
    }  
    private boolean redirectToSuccessUrl(HttpServletRequest req, HttpServletResponse resp) throws IOException {  
        WebUtils.redirectToSavedRequest(req, resp, successUrl);  
        return false;  
    }  
    private void saveRequestAndRedirectToLogin(HttpServletRequest req, HttpServletResponse resp) throws IOException {  
        WebUtils.saveRequest(req);  
        WebUtils.issueRedirect(req, resp, loginUrl);  
    }  
  
    private boolean login(HttpServletRequest req) {  
        String username = req.getParameter("username");  
        String password = req.getParameter("password");  
        try {  
            SecurityUtils.getSubject().login(new UsernamePasswordToken(username, password));  
        } catch (Exception e) {  
            req.setAttribute("shiroLoginFailure", e.getClass());  
            return false;  
        }  
        return true;  
    }  
    private boolean isLoginRequest(HttpServletRequest req) {  
        return pathsMatch(loginUrl, WebUtils.getPathWithinApplication(req));  
    }  
}   
......
```
onPreHandle主要流程：
1、首先判断是否已经登录过了，如果已经登录过了继续拦截器链即可；
2、如果没有登录，看看是否是登录请求，如果是get方法的登录页面请求，则继续拦截器链（到请求页面），否则如果是get方法的其他页面请求则保存当前请求并重定向到登录页面；
3、如果是post方法的登录页面表单提交请求，则收集用户名/密码登录即可，如果失败了保存错误消息到“shiroLoginFailure”并返回到登录页面；
4、如果登录成功了，且之前有保存的请求，则重定向到之前的这个请求，否则到默认的成功页面。
 
任意角色授权拦截器
Shiro提供roles拦截器，其验证用户拥有所有角色，没有提供验证用户拥有任意角色的拦截器。
```java
......
public class AnyRolesFilter extends AccessControlFilter {  
    private String unauthorizedUrl = "/unauthorized.jsp";  
    private String loginUrl = "/login.jsp";  
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {  
        String[] roles = (String[])mappedValue;  
        if(roles == null) {  
            return true;//如果没有设置角色参数，默认成功  
        }  
        for(String role : roles) {  
            if(getSubject(request, response).hasRole(role)) {  
                return true;  
            }  
        }  
        return false;//跳到onAccessDenied处理  
    }  
  
    @Override  
    protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception {  
        Subject subject = getSubject(request, response);  
        if (subject.getPrincipal() == null) {//表示没有登录，重定向到登录页面  
            saveRequest(request);  
            WebUtils.issueRedirect(request, response, loginUrl);  
        } else {  
            if (StringUtils.hasText(unauthorizedUrl)) {//如果有未授权页面跳转过去  
                WebUtils.issueRedirect(request, response, unauthorizedUrl);  
            } else {//否则返回401未授权状态码  
                WebUtils.toHttp(response).sendError(HttpServletResponse.SC_UNAUTHORIZED);  
            }  
        }  
        return false;  
    }  
}   
......
```
流程：
1、首先判断用户有没有任意角色，如果没有返回false，将到onAccessDenied进行处理；
2、如果用户没有角色，接着判断用户有没有登录，如果没有登录先重定向到登录；
3、如果用户没有角色且设置了未授权页面（unauthorizedUrl），那么重定向到未授权页面；否则直接返回401未授权错误码。

4.4 默认拦截器
Shiro内置了很多默认的拦截器，比如身份验证、授权等相关的。默认拦截器可以参考org.apache.shiro.web.filter.mgt.DefaultFilter中的枚举拦截器：
<table><tr><th>拦截器名</th><th>拦截器类</th><th>使用场景</th><th>说明（括号里的表示默认值）</th></tr><tr><td>authc</td><td>FormAuthenticationFilter</td><td>验证</td><td>基于表单的拦截器,如"/****=authc",如果没有登录会跳到相应的登录页面登录:主要属性:usernameParam:表单提交的用户名参数名(username):passwordParam:表单提交的密码参数名(password):rememberMeParam:表单提交的密码参数名(rememberMe):loginUrl:登录页面地址(/login.jsp):successUrl:登录成功后的默认重定向地址:failureKeyAttribute:登录失败后错误信息存储key(shiroLoginFailure)</td></tr><tr><td>authcBasic</td><td>BasicHttpAuthenticationFilter</td><td>验证</td><td>Basic HTTP身份验证拦截器，主要属性： applicationName：弹出登录框显示的信息（application）Basic HTTP身份验证拦截器,主要属性:applicationName,弹出登录框显示的信息(application)</td></tr><tr><td>logout</td><td>LogoutFilter</td><td>验证</td><td>退出拦截器,主要属性:redirectUrl:退出成功后重定向的地址(/)示例"/logout=logout"</td></tr><tr><td>user</td><td>UserFilter</td><td>验证</td><td>用户拦截器,用户已经身份验证/记住我登录的都可:示例/****=user</td></tr><tr><td>anon</td><td>AnonymousFilter</td><td>验证</td><td>匿名拦截器，即不需要登录即可访问:一般用于静态资源过滤:示例"/static/****=anon"</td></tr><tr><td>roles</td><td>RolesAuthorizationFilter</td><td>授权</td><td>角色授权拦截器,验证用户是否拥有所有角色:主要属性:loginUrl:登录页面地址(/login.jsp):unauthorizedUrl:未授权后重定向的地址:示例"/admin/****=roles[admin]"</td></tr><tr><td>perms</td><td>PermissionsAuthorizationFilter</td><td>授权</td><td>权限授权拦截器,验证用户是否拥有所有权限:属性和roles一样:示例"/user/**=perms["user:create"]"</td></tr><tr><td>port</td><td>PortFilter</td><td>授权</td><td>端口拦截器,主要属性:port(80):可以通过的端口:示例"/test= port[80]",如果用户访问该页面是非80,将自动将请求端口改为80并重定向到该80端口,其他路径/参数等都一样</td></tr><tr><td>rest</td><td>HttpMethodPermissionFilter</td><td>授权</td><td>rest风格拦截器,自动根据请求方法构建权限字符串(GET=read, POST=create,PUT=update,DELETE=delete,HEAD=read,TRACE=read,OPTIONS=read, MKCOL=create)构建权限字符串:示例"/users=rest[user]",会自动拼出"user:read,user:create,user:update,user:delete"权限字符串进行权限匹配(所有都得匹配,isPermittedAll)</td></tr><tr><td>ssl</td><td>SslFilter</td><td>授权</td><td>SSL拦截器,只有请求协议是https才能通过:否则自动跳转会https端口(443):其他和port拦截器一样SSL拦截器,只有请求协议是https才能通过:否则自动跳转会https端口(443):其他和port拦截器一样</td></tr><tr><td>noSessionCreation</td><td>NoSessionCreationFilter</td><td>其他</td><td>不创建会话拦截器,调用subject.getSession(false)不会有什么问题,但是如果subject.getSession(true)将抛出 DisabledSessionException异常</td></tr></table>
另外还提供了一个org.apache.shiro.web.filter.authz.HostFilter，即主机拦截器，比如其提供了属性：authorizedIps：已授权的ip地址，deniedIps：表示拒绝的ip地址；不过目前还没有完全实现，不可用。

5.参考文章
[跟我学Shiro](http://jinnianshilongnian.iteye.com/category/305053)