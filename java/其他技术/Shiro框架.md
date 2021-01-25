1. 在web.xml中配置shiro拦截器

    ~~~xml
    <!-- 
      1. 配置  Shiro 的 shiroFilter.  
      2. DelegatingFilterProxy 实际上是 Filter 的一个代理对象. 默认情况下, Spring 会到 IOC 容器中查找和 
      <filter-name> 对应的 filter bean. 也可以通过 targetBeanName 的初始化参数来配置 filter bean 的 id. 
    -->
    		
    <filter>  
        <filter-name>shiroFilter</filter-name>  
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
        <init-param>  
            <param-name>targetFilterLifecycle</param-name>  
            <param-value>true</param-value>  
        </init-param>  
    </filter>  
    <filter-mapping>  
        <filter-name>shiroFilter</filter-name>  
        <url-pattern>/*</url-pattern>  
    </filter-mapping> 
    ~~~

2. 在applicationContext.xml中配置shiro

    ~~~XML
    <!--  
     1. 配置 SecurityManager!
    -->     
    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <property name="cacheManager" ref="cacheManager"/>
        <property name="authenticator" ref="authenticator"></property>
        <property name="realm" ref="jdbcRealm"/>
    </bean>
    	
    <!--
     2. 配置 CacheManager. 
     2.1 需要加入 ehcache 的 jar 包及配置文件. 
    -->     
    <bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager">
        <property name="cacheManagerConfigFile" value="classpath:ehcache.xml"/> 
    </bean>
    	    
    <!-- 
      3. 配置 Realm 
      3.1 直接配置实现了 org.apache.shiro.realm.Realm 接口的 bean
    -->     
    <bean id="jdbcRealm" class="com.atguigu.shiro.realms.ShiroRealm">
        <property name="credentialsMatcher">
            <bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
                <property name="hashAlgorithmName" value="MD5"></property>
                <property name="hashIterations" value="1024"></property>
            </bean>
        </property>
    </bean>
    	    
    <!--  
      4. 配置 LifecycleBeanPostProcessor. 可以自定的来调用配置在 Spring IOC 容器中 shiro bean 的生命周期方法. 
    -->       
    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>
    	
    <!--  
      5. 启用 IOC 容器中使用 shiro 的注解. 但必须在配置了 LifecycleBeanPostProcessor 之后才可以使用. 
    -->     
    <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
          depends-on="lifecycleBeanPostProcessor"/>
    <bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
        <property name="securityManager" ref="securityManager"/>
    </bean>
    	
    <!--  
      6. 配置 ShiroFilter. 
      6.1 id 必须和 web.xml 文件中配置的 DelegatingFilterProxy 的 <filter-name> 一致.
        若不一致, 则会抛出: NoSuchBeanDefinitionException. 因为 Shiro 会来 IOC 容器中查找和 <filter-name> 名字对应的 filter bean.
         -->     
    <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <property name="securityManager" ref="securityManager"/>
        <property name="loginUrl" value="/login.jsp"/>
        <property name="successUrl" value="/list.jsp"/>
        <property name="unauthorizedUrl" value="/unauthorized.jsp"/>
    
        <!--  
              配置哪些页面需要受保护. 
              以及访问这些页面需要的权限. 
              1). anon 可以被匿名访问
              2). authc 必须认证(即登录)后才可能访问的页面. 
              3). logout 登出.
              4). roles 角色过滤器
             -->
        <property name="filterChainDefinitions">
            <value>
                /login.jsp = anon
                /shiro/login = anon
                /shiro/logout = logout
    
                /user.jsp = roles[user]
                /admin.jsp = roles[admin]
    
                # everything else requires authentication:
                /** = authc
            </value>
        </property>
    </bean>
    ~~~

    

3. 自定义Realm

    ~~~JAVA
    import java.util.HashSet;
    import java.util.Set;
    import org.apache.shiro.authc.AuthenticationException;
    import org.apache.shiro.authc.AuthenticationInfo;
    import org.apache.shiro.authc.AuthenticationToken;
    import org.apache.shiro.authc.LockedAccountException;
    import org.apache.shiro.authc.SimpleAuthenticationInfo;
    import org.apache.shiro.authc.UnknownAccountException;
    import org.apache.shiro.authc.UsernamePasswordToken;
    import org.apache.shiro.authz.AuthorizationInfo;
    import org.apache.shiro.authz.SimpleAuthorizationInfo;
    import org.apache.shiro.crypto.hash.SimpleHash;
    import org.apache.shiro.realm.AuthorizingRealm;
    import org.apache.shiro.subject.PrincipalCollection;
    import org.apache.shiro.util.ByteSource;
    
    public class ShiroRealm extends AuthorizingRealm {
    
        //认证
        @Override
        protected AuthenticationInfo doGetAuthenticationInfo(
            AuthenticationToken token) throws AuthenticationException {
            System.out.println("[FirstRealm] doGetAuthenticationInfo");
    
            //1. 把 AuthenticationToken 转换为 UsernamePasswordToken 
            UsernamePasswordToken upToken = (UsernamePasswordToken) token;
    
            //2. 从 UsernamePasswordToken 中来获取 username
            String username = upToken.getUsername();
    
            //3. 调用数据库的方法, 从数据库中查询 username 对应的用户记录
            System.out.println("从数据库中获取 username: " + username + " 所对应的用户信息.");
    
            //4. 若用户不存在, 则可以抛出 UnknownAccountException 异常
            if("unknown".equals(username)){
                throw new UnknownAccountException("用户不存在!");
            }
    
            //5. 根据用户信息的情况, 决定是否需要抛出其他的 AuthenticationException 异常. 
            if("monster".equals(username)){
                throw new LockedAccountException("用户被锁定");
            }
    
            //6. 根据用户的情况, 来构建 AuthenticationInfo 对象并返回. 通常使用的实现类为: SimpleAuthenticationInfo
            //以下信息是从数据库中获取的.
    
            //1). principal: 认证的实体信息. 可以是 username, 也可以是数据表对应的用户的实体类对象. 
            Object principal = username;
            //2). credentials: 密码. 
            Object credentials = "fc1709d0a95a6be30bc5926fdb7f22f4";
    
            //3). realmName: 当前 realm 对象的 name. 调用父类的 getName() 方法即可
            String realmName = getName();
            //4). 盐值. 
            ByteSource credentialsSalt = ByteSource.Util.bytes(username);
    
            //返回认证信息
            SimpleAuthenticationInfo info = null; //new SimpleAuthenticationInfo(principal, credentials, realmName);
            info = new SimpleAuthenticationInfo(principal, credentials, credentialsSalt, realmName);
            return info;
        }
    
        public static void main(String[] args) {
            String hashAlgorithmName = "MD5";
            Object credentials = "123456";
            Object salt = ByteSource.Util.bytes("user");;
            int hashIterations = 1024;
    
            Object result = new SimpleHash(hashAlgorithmName, credentials, salt, hashIterations);
            System.out.println(result);
        }
    
    
        /**
    	  * 权限认证
    	  */
        @Override
        protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
    
            UserAuthService shiroFactory = UserAuthServiceServiceImpl.me();
            //1. 从 PrincipalCollection 中来获取登录用户的信息
            ShiroUser shiroUser = (ShiroUser) principals.getPrimaryPrincipal();
            List<Integer> roleList = shiroUser.getRoleList();
    
            Set<String> permissionSet = new HashSet<>();
            Set<String> roleNameSet = new HashSet<>();
            //2. 利用登录的用户的信息来用户当前用户的角色或权限(可能需要查询数据库)
            for (Integer roleId : roleList) {
                List<String> permissions = shiroFactory.findPermissionsByRoleId(roleId);
                if (permissions != null) {
                    for (String permission : permissions) {
                        if (ToolUtil.isNotEmpty(permission)) {
                            permissionSet.add(permission);
                        }
                    }
                }
                String roleName = shiroFactory.findRoleNameByRoleId(roleId);
                roleNameSet.add(roleName);
            }
            //3. 创建 SimpleAuthorizationInfo, 并设置其 reles 属性.
            SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
            info.addStringPermissions(permissionSet);
            info.addRoles(roleNameSet);
            return info;
        }
    }
    
    ~~~

4. 登录controller

    ~~~JAVA
    @RequestMapping("/login")
    public String login(@RequestParam("username") String username, 
                        @RequestParam("password") String password){
    
        Subject currentUser = SecurityUtils.getSubject();
        if (!currentUser.isAuthenticated()) {
            // 把用户名和密码封装为 UsernamePasswordToken 对象
            UsernamePasswordToken token = new UsernamePasswordToken(username, password);
    
            token.setRememberMe(true);
            try {
                // 执行登录. 
                currentUser.login(token);
            } 
            catch (AuthenticationException ae) {
                System.out.println("登录失败: " + ae.getMessage());
            }
        }
        return "redirect:/login.jsp";
    }
    ~~~



**springboot配置shiro**

~~~XML
 <!--shiro依赖和缓存-->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>${shiro.version}</version>
    <exclusions>
        <exclusion>
            <artifactId>slf4j-api</artifactId>
            <groupId>org.slf4j</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>${shiro.version}</version>
</dependency>
<!-- <dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-ehcache</artifactId>
    <version>${shiro.version}</version>
    <exclusions>
        <exclusion>
            <artifactId>slf4j-api</artifactId>
            <groupId>org.slf4j</groupId>
        </exclusion>
    </exclusions>
</dependency> -->

~~~



~~~JAVA
@Configuration
public class ShiroConfiguration {

    /**
     * ShiroFilterFactoryBean 处理拦截资源文件问题。
     * 注意：单独一个ShiroFilterFactoryBean配置是或报错的，以为在
     * 初始化ShiroFilterFactoryBean的时候需要注入：SecurityManager
     *
     * Filter Chain定义说明 1、一个URL可以配置多个Filter，使用逗号分隔 2、当设置多个过滤器时，全部验证通过，才视为通过
     * 3、部分过滤器可指定参数，如perms，roles
     *
     */
    @Bean
    public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();

        // 必须设置 SecurityManager
        shiroFilterFactoryBean.setSecurityManager(securityManager);

        // 如果不设置默认会自动寻找Web工程根目录下的"/login.jsp"页面
        shiroFilterFactoryBean.setLoginUrl("/login");
        // 登录成功后要跳转的链接
        shiroFilterFactoryBean.setSuccessUrl("/index");
        // 未授权界面;
        shiroFilterFactoryBean.setUnauthorizedUrl("/403");

        //自定义拦截器
        Map<String, Filter> filtersMap = new LinkedHashMap<String, Filter>();
        //限制同一帐号同时在线的个数。
        //filtersMap.put("kickout", kickoutSessionControlFilter());
        shiroFilterFactoryBean.setFilters(filtersMap);

        // 权限控制map.
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
        // 配置不会被拦截的链接 顺序判断
        // 配置退出过滤器,其中的具体的退出代码Shiro已经替我们实现了
        // 从数据库获取动态的权限
        // filterChainDefinitionMap.put("/add", "perms[权限添加]");
        // <!-- 过滤链定义，从上向下顺序执行，一般将 /**放在最为下边 -->:这是一个坑呢，一不小心代码就不好使了;
        // <!-- authc:所有url都必须认证通过才可以访问; anon:所有url都都可以匿名访问-->
        //logout这个拦截器是shiro已经实现好了的。
        // 从数据库获取
        /*List<SysPermissionInit> list = sysPermissionInitService.selectAll();

        for (SysPermissionInit sysPermissionInit : list) {
            filterChainDefinitionMap.put(sysPermissionInit.getUrl(),
                    sysPermissionInit.getPermissionInit());
        }*/

        shiroFilterFactoryBean
                .setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;
    }

    @Bean
    public SecurityManager securityManager() {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        // 设置realm.
        securityManager.setRealm(myShiroRealm());
        // 自定义缓存实现 使用redis
        //securityManager.setCacheManager(cacheManager());
        // 自定义session管理 使用redis
        //securityManager.setSessionManager(sessionManager());
        //注入记住我管理器;
        securityManager.setRememberMeManager(rememberMeManager());
        return securityManager;
    }

    public MyShiroRealm myShiroRealm(){
        return new MyShiroRealm();
    }

    /**
     * cookie对象;
     * @return
     */
    public SimpleCookie rememberMeCookie(){
       //这个参数是cookie的名称，对应前端的checkbox的name = rememberMe
       SimpleCookie simpleCookie = new SimpleCookie("rememberMe");
       //<!-- 记住我cookie生效时间30天 ,单位秒;-->
       simpleCookie.setMaxAge(2592000);
       return simpleCookie;
    }

    /**
     * cookie管理对象;记住我功能
     * @return
     */
    public CookieRememberMeManager rememberMeManager(){
       CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
       cookieRememberMeManager.setCookie(rememberMeCookie());
       //rememberMe cookie加密的密钥 建议每个项目都不一样 默认AES算法 密钥长度(128 256 512 位)
       cookieRememberMeManager.setCipherKey(Base64.decode("3AvVhmFLUs0KTA3Kprsdag=="));
       return cookieRememberMeManager;
    }

}

~~~

