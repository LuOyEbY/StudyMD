### SpringBoot项目实现是初始化操作的三种方式

##### 1.使用@PostStruct注解

###### 该注解也是在项目过程中执行初始化的作用。比ApplicationRunner要快。

```java
// An highlighted block
package config;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import javax.annotation.PostConstruct;

/**
 * @author Damon-z
 */
@SpringBootApplication
public class ProjectApplication {
    private static final Logger log = LoggerFactory.getLogger(ProjectApplication .class);
    public static void main(String[] args) {
        SpringApplication.run(ProjectApplication.class, args);
    }

    @PostConstruct
    public void init() {
        log.info("初始化方法开始PostConstruct");
       //Util.init();
        log.info("初始化方法结束PostConstruct");
    }

}
```

##### 2.ApplicationRunner

###### 对于springboot项目我们可以采用实现ApplicationRunner类来进行初始化，方法也很简单，直接在run方法中写下你要执行的初始化代码就行。该方法比@PostConstruct注解要慢。

```java
// An highlighted block
package config;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;


/**
 * @author Damon-z
 * @date 2019/8/6 17:30
 */
@Component
public class InitConfig implements ApplicationRunner {
    private static final Logger log = LoggerFactory.getLogger(InitConfig.class);
    @Override
    public void run(ApplicationArguments args) {
        log.info("项目启动初始化开始ApplicationRunner");
        Util.init();
        log.info("项目启动初始化结束ApplicationRunner");
    }
}
```



##### 3.InitializingBean

###### 该方法速度与@PostConstruct一样，实现方法也很简单，

```java
// An highlighted block
package config;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.InitializingBean;

/**
 * @author Damon-z
 * @date 2019/8/11 20:34
 */
public class InitBean implements InitializingBean {
    private static final Logger log = LoggerFactory.getLogger(InitConfig.class);
    @Override
    public void afterPropertiesSet() {
        log.info("项目启动初始化开始InitializingBean");
        Util.init();
        log.info("项目启动初始化结束InitializingBean");
    }
}
```



##### 总结：

###### 以上几种方式，InitializingBean 是不用springboot就能实现的。但是如果我们要在初始化过程中使用静态static的方法块并调用静态变量时，应采用ApplicationRunner。

