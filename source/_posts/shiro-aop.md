---
title: 使用Shiro 导致AOP切面失效
date: 2020-04-27 20:38:03
tags:
    - Java 
categories:
    - Java
---

{% blockquote %}
如题，当Springboot集成Shiro时。
Shiro认证ShiroRealm 中引入Service的同时，AOP对该Service有命中切面。会导致AOP失效

你若是找不到坚持下去的理由，那么你就找一个重新开始的理由，生活本来就这么简单。
{% endblockquote %}
##### AOP失效问题

```java
public class MyShiroRealm extends AuthorizingRealm {
    @Autowired
    UserService userService;
}
```

##### 解决方法。
    
    网上说是Shiro与Aspect的冲突。
    给Shiro认证器单独写一个Service。 
    
    有别的解决方式，请联系我。共同进步。
    
    找到了。shiroRealm 中引入Service时加 @Lazy注解。
