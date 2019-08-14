title: SpringBoot中使用Shiro自动装配的坑
author: 26186
tags:
  - java
  - SpringBoot
  - Shiro
categories:
  - java
date: 2019-08-14 16:11:00
---
因前段时间通过`jasypt`对配置文件做了加密处理，今天突然找我说Shiro配置的JWT免密登录接口都不能访问了。what？

第一反应是增加的`jasypt-spring-boot-starter`干了什么蠢事，伴随着心虚去除了`jasypt-spring-boot-starter`，问题依旧存在。在问题还没扩散前通知伙伴，这锅不是我的。此时感觉神清气爽。

第二反应，百度`shiro.filterRule`无效，果然一堆类似的问题。最终定位解决方案是`anon`配置需要在前面。我们配置如下：
```config
spring.jwt.shiro.filterRule[//]=anon
spring.jwt.shiro.filterRule[/**]=jwt[del,add]
```
> `[//]`为配置的 url 路径，此处省略。
> 实际项目配置了很多`anon`。

`anon`是在前面呀！那可能就是`key-value`的`Map`导致了无序。找到`ShiroConfig`查看，自动装配使用了`LinkedHashMap`，那就是有序的，表面能看到的代码都没有发现问题。

第三反应，大招 Debug 开始，启动项目进行调试。
![upload successful](/images/pasted-11.png)
问题已经找到自动装配把顺序装配错了，自动装配坑啊！！！
修改代码：
```java
@Bean("shiroFilter")
public ShiroFilterFactoryBean factory(DefaultWebSecurityManager securityManager) {
  ShiroFilterFactoryBean factoryBean = new ShiroFilterFactoryBean();
    
  // 添加自己的过滤器并且取名为jwt
  Map<String, Filter> filterMap = new HashMap<>();
  filterMap.put("jwt", new JWTFilter(shiroProps.getLogin()));
  factoryBean.setFilters(filterMap);
  factoryBean.setSecurityManager(securityManager);

  factoryBean.setUnauthorizedUrl(shiroProps.getUnauthorizedUrl());
  Map<String, String> oldMap = shiroProps.getFilterRule();
  // 自动装配的配置项排序不正确，anon需要排在最前面。
  List<Map.Entry<String,String>> lstEntry=new ArrayList<>(oldMap.entrySet());
  Collections.sort(lstEntry,new Comparator<Entry<String,String>>() {
    @Override
    public int compare(Entry<String,String> o1, Entry<String,String> o2) {
      return o1.getValue().compareTo(o2.getValue());
    }
  });
  Map<String, String> newMap = new LinkedHashMap<String, String>();
  for (Entry<String, String> entry : lstEntry) {
    newMap.put(entry.getKey(), entry.getValue());
  }
  
  factoryBean.setFilterChainDefinitionMap(newMap);
  return factoryBean;
    }
```
问题解决。