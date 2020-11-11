# Spring boot 避坑指南

##### 1.配置总出错，搞清楚加载顺序

```java
1.spring boot的配置文件
	a:SpringBoot使用一个全局的配置文件，且配置文件名是固定的。配置文件的作用是来修改SpringBoot自动配置的默认值。
  b:可以使用application.properties,也可以使用application.yml
  c:由于YAML格式紧凑且可读性高，所以，SpringBoot支持并推荐使用YAML配置文件
  d:如果两种配置文件同事存在的时候，默认优先使用.properties配置文件(但是，一般不会这么做)
2.spring boot 配置文件优先加载顺序
  a:file:./config/ 			-->优先级最高(项目根路径下的config)
  b:file:./        			-->优先级第二(项目根路径下)
  c:classpath:/config/ 	-->优先级第三(项目resources/config下)
  d:classpath:/    			-->优先级第四(项目resources根目录下)
3.遵循的规则
  a:高优先级的会覆盖低优先级的
  b:多个文件是互补的，取的是多个文件的并集
4.spring boot多环境配置
  a:多环境下使用spring.profile.active可以指定配置文件
  b:使用占位符${spring.profiles.active},在启动命令中指定配置文件
```

##### 2.定时任务不定时了，到底是怎么了？

```java
1.
```



