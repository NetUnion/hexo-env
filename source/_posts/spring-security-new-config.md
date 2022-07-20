---
title: Spring Security 新版配置文件写法
date: 2022-07-20 17:02:14
tags: [Java, Spring Boot, Spring Security]
description: Spring Security 在 5.7.0 及之后的版本中启用的新的配置文件写法。本文提供了一些示例。
---

Spring Security 在 5.7.0 及之后的版本中启用的新的配置文件写法。弃用了之前的 `WebSecurityConfigurerAdapter`，官方也推出了一篇博文来说明新的配置写法：[Spring Security without the WebSecurityConfigurerAdapter](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter) 但这篇文章还是没有把一些问题说清楚，在查阅了一些文档和 stackoverflow 帖子后，笔者总结了一些在 `NuManager` 项目中的写法，供读者参考。

## HttpSecurity

在 5.4 版本中新增了通过创建 `SecurityFilterChain`  bean 来配置 `HttpConfig` 的写法，现在只剩这种写法了。记得要返回对应 bean 的 `build` 方法。

```java
@Bean
    public SecurityFilterChain httpSecurity(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/index").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin() //启用表单登录
                .defaultSuccessUrl("/index")
                .and()
                .oauth2Login() //启用oauth2登录
                .defaultSuccessUrl("/index")
                .and()
                .oauth2Client()
                .and()
                .csrf().disable() //禁用csrf
                .sessionManagement()
                .maximumSessions(1); //保证一个用户只能存在一个会话。
        return http.build();
    }
```

## 基于外部数据库的帐号密码存储配置

在用户对应的 pojo 中，要改下普通的权限的 Getter 方法

```java
 @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        String[] authorities = this.authority.split(",");
        List<SimpleGrantedAuthority> simpleGrantedAuthorities = new ArrayList<>();
        for (String role : authorities) {
            simpleGrantedAuthorities.add(new SimpleGrantedAuthority(role));
        }
        return simpleGrantedAuthorities;
    }
```

要有一个 `DBUserDetailsService.class` 来获取帐号密码

```java
@Service
public class DBUserDetailsService implements UserDetailsService {
    @Autowired
    private UserMapper userMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return userMapper.getByUsername(username);
    }
}
```

在配置中要设置密码加密算法

```java
@Bean
    // 设置密码加密方式
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

然后配置新的认证管理器

```java
@Bean
public AuthenticationManager authenticationManager(AuthenticationConfiguration authenticationConfiguration) throws Exception {
    return authenticationConfiguration.getAuthenticationManager();
}
```

这里会自动调用 `DBUserDetailsService`。
