# SpringSecurity

Spring Boot 3 已经升级到 **Spring Security 6**

`authorizeRequests()` 返回的不再是 `ExpressionUrlAuthorizationConfigurer.ExpressionInterceptUrlRegistry`，
而变成了 `AuthorizeHttpRequestsConfigurer.AuthorizedUrl`；
**`antMatchers()` 方法被彻底移除**，

## OAuth2

密码模式和授权码授权模式的区别：

密码授权和授权码授权最大的区别是授权码模式多了一个授权码环节

四种授权模型

- 授权码模式
- 隐藏模式
- 密码模式
- 凭证式模式

## 面试题

### 关于 Spring Security 6

#### 基础概念与架构
1. **请简述 Spring Security 的核心组件，例如 Authentication、UserDetails 等，以及它们在 Spring Security 6 中的作用。**
   - **Authentication**：表示用户的认证信息，通常包含用户的用户名、密码、权限等信息。在 Spring Security 中，`Authentication` 是一个接口，常用的实现类是 `UsernamePasswordAuthenticationToken`。
   - **UserDetails**：表示用户的具体信息，通常包含用户名、密码、权限等。`UserDetails` 是一个接口，常用的实现类是 `User`。`UserDetailsService` 是一个接口，用于加载用户信息，通常在认证过程中被调用。
   - **GrantedAuthority**：表示用户的权限，通常是一个接口，常用的实现类是 `SimpleGrantedAuthority`。
   - **PasswordEncoder**：用于密码加密，常用的实现类是 `BCryptPasswordEncoder`，用于将用户输入的明文密码加密后与数据库中存储的加密密码进行比对。

2. **在 Spring Security 6 中，如何配置自定义的用户认证逻辑？请举例说明。**
   - 在 Spring Security 6 中，可以通过实现 `UserDetailsService` 接口来自定义用户认证逻辑。以下是一个简单的例子：
     ```java
     @Service
     public class UserDetailsServiceImpl implements UserDetailsService {
     
     	@Autowired
     	private SysUserService sysUserService;
     	@Autowired
     	private RedisTemplate<String, String> redisTemplate;
     	
     	private static final List<Object> KEYS = new ArrayList<Object>();
     	static {
     		KEYS.add(PubCodeConstants.PASSWORD.EDIT_FIRST.getCode());
     		KEYS.add(PubCodeConstants.PASSWORD.CYCLE_EDIT.getCode());
     	}
     
     	@Override
     	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
     		SysUserVo userVo = sysUserService.loadUserByUsername(username);
     		List<Object> list = redisTemplate.boundHashOps(CommonConstants.SYSTEM_PUBLIC_CODE).multiGet(KEYS);
     
     		Object eidtFirst = list.get(0);
     		LocalDateTime last = sysUserService.lastModifyTime(username);
     		// 判断初次登录是否强制修改密码
     		if (eidtFirst != null && PubCodeConstants.PASSWORD.EDIT_FIRST.getValue().equals(String.valueOf(eidtFirst))) {
     			if (last == null) {
     				throw new BadCredentialsException("FIRST");
     			}
     		}
     		if (last == null) {
     			last = userVo.getCreateTime();
     		}
     		// 判断周期修改密码(单位：天),判断密码是否过期
     		Duration duration = Duration.between(last, LocalDateTime.now());
     		if (list.get(1) != null) {
     			Integer cycle = Integer.valueOf(String.valueOf(list.get(1)));
     			if (duration.toDays() > cycle) {// 未修改密码，被锁定
     				throw new BadCredentialsException("LOCKED");
     			}
     		}	
     		return new UserDetailsImpl(userVo);
     	}
     }
     ```
   
3. **你了解 Spring Security 6 中的过滤器链吗？请说明其中一些关键过滤器的作用和执行顺序。**
   - Spring Security 使用过滤器链来处理安全相关的操作。以下是一些关键过滤器及其作用：
     - **SecurityContextPersistenceFilter**：负责在请求开始时恢复安全上下文（`SecurityContext`），并在请求结束时清除它。
     - **LogoutFilter**：处理用户注销逻辑。
     - **UsernamePasswordAuthenticationFilter**：处理基于用户名和密码的认证请求。
     - **BasicAuthenticationFilter**：处理基于 HTTP Basic 认证的请求。
     - **ExceptionTranslationFilter**：捕获安全相关的异常，并将它们转换为 HTTP 响应。
     - **FilterSecurityInterceptor**：负责对请求进行授权检查。
   - 这些过滤器的执行顺序通常是由 Spring Security 的配置决定的，可以通过 `HttpSecurity` 配置类来调整。

#### 功能实现与应用
4. **如何在 Spring Security 6 中实现基于角色的访问控制？请给出代码示例。**
   - 在 Spring Security 6 中，可以通过 `HttpSecurity` 配置类来实现基于角色的访问控制。以下是一个简单的例子：
     ```java
     @Configuration
     @EnableWebSecurity
     public class SecurityConfig extends WebSecurityConfigurerAdapter {
         @Override
         protected void configure(HttpSecurity http) throws Exception {
             http
                 .authorizeRequests()
                 .antMatchers("/admin/**").hasRole("ADMIN") // 只有角色为 ADMIN 的用户可以访问
                 .antMatchers("/user/**").hasRole("USER") // 只有角色为 USER 的用户可以访问
                 .anyRequest().authenticated() // 其他请求需要认证
                 .and()
                 .formLogin()
                 .and()
                 .logout();
         }
     }
     ```

5. **请说明如何在 Spring Security 6 中配置密码加密存储，例如使用 BCrypt 算法。**
   
   - 在 Spring Security 6 中，可以通过配置 `PasswordEncoder` 来实现密码加密存储。以下是一个简单的例子：
     ```java
     @Configuration
     @EnableWebSecurity
     public class SecurityConfig extends WebSecurityConfigurerAdapter {
         @Bean
         public PasswordEncoder passwordEncoder() {
             return new BCryptPasswordEncoder();
         }
         
         @Override
         protected void configure(AuthenticationManagerBuilder auth) throws Exception {
             auth.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder());
         }
     }
     ```
   
6. **在一个实际的项目中，如果需要对某些敏感接口进行 IP 白名单限制，你会如何利用 Spring Security 6 来实现？**
   - 可以通过自定义一个过滤器来实现 IP 白名单限制。以下是一个简单的例子：
     ```java
     @Component
     public class IpWhiteListFilter extends OncePerRequestFilter {
         private final List<String> whiteList = Arrays.asList("192.168.1.1", "192.168.1.2");
         
         @Override
         protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
             String remoteAddr = request.getRemoteAddr();
             if (!whiteList.contains(remoteAddr)) {
                 response.setStatus(HttpServletResponse.SC_FORBIDDEN);
                 return;
             }
             filterChain.doFilter(request, response);
         }
     }
     ```

#### 安全特性与优化
7. **Spring Security 6 对于防止常见的安全漏洞（如 CSRF 攻击、XSS 攻击）有哪些内置的机制？如何启用和配置这些机制？**
   - **CSRF 攻击**：Spring Security 默认启用了 CSRF 保护。可以通过以下方式启用或禁用 CSRF 保护：
     ```java
     @Override
     protected void configure(HttpSecurity http) throws Exception {
         http
             .csrf().disable() // 禁用 CSRF 保护
             .csrf().csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse()) // 自定义 CSRF Token 存储方式
             .and()
             .formLogin();
     }
     ```
   - **XSS 攻击**：Spring Security 本身不直接处理 XSS 攻击，但可以通过配置 `HttpSecurity` 的 `xssProtection` 方法来启用浏览器的 XSS 保护机制：
     
     ```java
     @Override
     protected void configure(HttpSecurity http) throws Exception {
         http
             .headers()
             .xssProtection()
             .and()
             .formLogin();
     }
     ```
   
8. **请谈谈你对 Spring Security 6 中的会话管理的理解，如何配置无状态会话（stateless session）？**
   - 无状态会话（stateless session）是指不依赖于服务器端的会话信息，通常用于 RESTful API。在 Spring Security 6 中，可以通过以下方式配置无状态会话：
     ```java
     @Override
     protected void configure(HttpSecurity http) throws Exception {
         http
             .sessionManagement()
             .sessionCreationPolicy(SessionCreationPolicy.STATELESS) // 配置无状态会话
             .and()
             .formLogin()
             .and()
             .csrf().disable(); // 无状态会话通常需要禁用 CSRF 保护
     }
     ```

9. **在使用 Spring Security 6 时，如何优化性能，例如减少认证过程中的资源消耗？**
   - 可以通过以下方式优化性能：
     - **缓存用户信息**：使用缓存（如 Redis）来存储用户信息，减少对数据库的访问。
     - **异步认证**：对于一些复杂的认证逻辑，可以使用异步认证来减少主线程的阻塞。
     - **优化过滤器链**：只在必要时启用某些过滤器，减少不必要的处理。

### 关于 OAuth 2.0

#### 基础理论
10. **请简述 OAuth 2.0 的四种授权模式（授权码模式、隐式授权模式、密码模式、客户端凭证模式），并分别说明它们的适用场景。**
    - **授权码模式（Authorization Code）**：这是最常用的一种模式，适用于客户端有后端服务的情况。用户在授权服务器上授权后，授权服务器会返回一个授权码，客户端再用授权码换取访问令牌。
    - **隐式授权模式（Implicit）**：适用于客户端没有后端服务的情况（如单页应用）。用户在授权服务器上授权后，授权服务器直接返回访问令牌。
    - **密码模式（Resource Owner Password Credentials）**：适用于客户端是可信的，且用户愿意将用户名和密码直接提供给客户端的情况。客户端使用用户的用户名和密码直接向授权服务器换取访问令牌。
    - **客户端凭证模式（Client Credentials）**：适用于客户端是可信的，且不需要用户授权的情况。客户端使用自己的客户端凭证（client_id 和 client_secret）向授权服务器换取访问令牌。

11. **在 OAuth 2.0 的授权码模式中，授权码的作用是什么？它是如何保证安全性的？**
    - **授权码的作用**：授权码是一个临时的凭证，用于在用户授权后，客户端换取访问令牌。它是一个中间凭证，用于防止客户端直接获取用户的用户名和密码。
    - **安全性**：授权码是短暂有效的，通常在几分钟内过期。授权码只能使用一次，使用后会被授权服务器标记为已使用。

```json
{"@class":"java.util.Collections$UnmodifiableMap","java.security.Principal":{"@class":"org.springframework.security.authentication.UsernamePasswordAuthenticationToken","authorities":["java.util.Collections$UnmodifiableRandomAccessList",[{"@class":"org.springframework.security.core.authority.SimpleGrantedAuthority","authority":"ROLE_ADMIN=role_admin"},{"@class":"org.springframework.security.core.authority.SimpleGrantedAuthority","authority":"SYSTEM_DEFAULT_ROLE_TO_USER=SYSTEM_DEFAULT_ROLE_TO_USER"}]],"details":null,"authenticated":true,"principal":{"@class":"com.littlelee.base.auth.security.UserDetailsImpl","userId":"user_admin","username":"admin","password":null,"status":"0","roleVos":["java.util.ArrayList",[{"@class":"com.littlelee.base.common.model.vo.SysRoleVo","roleId":"role_admin","roleCode":"ROLE_ADMIN","roleName":"管理员角色","createTime":[2018,10,16,9,47,54],"modifyTime":[2025,5,16,17,6,34],"delFlag":"0","appId":"base"}]],"deptId":null,"dataScope":null,"enabled":null,"authorities":["java.util.ArrayList",[{"@class":"org.springframework.security.core.authority.SimpleGrantedAuthority","authority":"ROLE_ADMIN=role_admin"},{"@class":"org.springframework.security.core.authority.SimpleGrantedAuthority","authority":"SYSTEM_DEFAULT_ROLE_TO_USER=SYSTEM_DEFAULT_ROLE_TO_USER"}]],"accountNonExpired":true,"accountNonLocked":true,"credentialsNonExpired":true},"credentials":null}}
```



> https://blog.csdn.net/u013737132/article/details/134024381
>
> 开源微服务商城项目 youlai-mall：https://gitee.com/youlaitech/youlai-mall

Spring Security OAuth2 的最终版本是2.5.2，并于2022年6月5日正式宣布停止维护。Spring 官方为此推出了新的替代产品，即 **Spring Authorization Server**。然而，出于安全考虑，Spring Authorization Server **不再支持密码模式**，因为密码模式要求客户端直接处理用户的密码。但对于受信任的第一方系统(自有APP和管理系统等)，许多情况下需要使用密码模式。在这种情况下，需要在 Spring Authorization Server 的基础上扩展密码模式的支持。本文基于开源微服务商城项目 youlai-mall、Spring Boot 3 和 Spring Authorization Server 1.1 版本，演示了如何扩展密码模式，以及如何将其应用于 Spring Cloud 微服务实战。


- 令牌发放记录表: [oauth2-authorization-schema.sql](https://github.com/spring-projects/spring-authorization-server/blob/main/oauth2-authorization-server/src/main/resources/org/springframework/security/oauth2/server/authorization/oauth2-authorization-schema.sql)
- 授权记录表: [oauth2-authorization-consent-schema.sql](https://blog.csdn.net/u013737132/article/details/oauth2-authorization-consent-schema.sql)
- 客户端信息表: [oauth2-registered-client-schema.sql](https://github.com/spring-projects/spring-authorization-server/blob/main/oauth2-authorization-server/src/main/resources/org/springframework/security/oauth2/server/authorization/client/oauth2-registered-client-schema.sql)
