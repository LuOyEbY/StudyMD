# SpringBoot脚手架的搭建

#### 集成MybatisPius实现数据持久化

```java
1.在pom.xml中引入相关的jar包依赖
  			<!-- Mybatis Plus集成 -->
        <!-- Mysql支持 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <!-- mybatis支持 -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.1</version>
        </dependency>
        <!-- mybatis plus支持 -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.3.1</version>
        </dependency>
2.实现XxxMapper接口，通过此接口来操作数据持久化
3.对XxxDO实体进行注解定义，如：数据库表名，字段的定义
4.如需修改Plus默认配置，需要实现mybatisPlusConfig类
5.如需要自定一个方法，需实现XxxMapper.xml来定义自定义接口
```



#### MybatisPlus实现乐观锁以及使用规则

```java
实现乐观锁:
	1.数据库字段中带上version字段，并加上@version注解。
   	/**
     * 版本号
     */
    @Version
    @TableField(fill = FieldFill.INSERT)
    private Long version;
	2.编写MybatisPlusConfig配置类，创建OptimistiLockerInterceptor@Bean类，开启乐观锁的使用。
     /**
     * 乐观锁的配置及使用规则
     * 1.如果更新数据中不带有version字段：不使用乐观锁，并且version不会累加
     * 2.如果更新字段中带有version字段。但是与数据库中不一致，更新失败
     * 3.如果带有version，并且与数据库中一致，则更新成功，并且version会累加
     *
     * @return
     */
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor(){
        return new OptimisticLockerInterceptor();
    }

乐观锁的使用规则:
	1.如果更新数据的字段中不带version字段，则不会使用乐观锁，并且version不会累加。
	2.如果更新数据带有version字段，但是与数据库中的值不一致，则更新失败。
	3.如果更新数据带有version字段，并且与数据库中的值一致，则更新成功，并且version自动增加+1。
```



#### MybatisPlus分页插件使用及配置

```java
实现分页插件：
	1.编写MybatisPlusConfig配置类，创建PaginationInterceptor@Bean类，开启分页插件的使用。
  	/**
     * 分页功能的实现
     * @return
     */
    @Bean
    public PaginationInterceptor paginationInterceptor(){
        PaginationInterceptor paginationInterceptor = new PaginationInterceptor();

        //数据库类型配置
        paginationInterceptor.setDbType(DbType.MYSQL);

        return paginationInterceptor;
    }
  
  2.在分页查询中使用MyBatisPlus集成Page类和QueryWrapper类，然后调用selectPage()方法使用分页插件，完成分页功能。	
  	@Override
    public PageResult<List<UserDTO>> query(PageQuery<UserQueryDTO> pageQuery) {
				
  			//使用Mybatis分页辅助类
        Page page = new Page(pageQuery.getPageNo(), pageQuery.getPageSize());

        UserDO query = new UserDO();
        BeanUtils.copyProperties(pageQuery.getQuery(), query);
        //TODO 如果属性不一致，需要做特殊处理
        QueryWrapper queryWrapper = new QueryWrapper(query);

        //调用Mybatis分页查询方法
        IPage<UserDO> userDOIPage = userMapper.selectPage(page, queryWrapper);

        //结果解析
        PageResult pageResult = new PageResult();
        pageResult.setPageNo((int) userDOIPage.getCurrent());
        pageResult.setPageSize((int) userDOIPage.getSize());
        pageResult.setTotal(userDOIPage.getTotal());
        pageResult.setPageNum(userDOIPage.getPages());

        List<UserDTO> collect = Optional.ofNullable(userDOIPage.getRecords())
                .map(List::stream)
                .orElseGet(Stream::empty)
                .map(userDO -> {
                    UserDTO userDTO = new UserDTO();
                    BeanUtils.copyProperties(userDO, userDTO);
                    return userDTO;
                }).collect(Collectors.toList());

        pageResult.setData(collect);

        return pageResult;
    }
```



#### 实现自动更新系统级字段

```java
实现方法：
	1.编写公共的源数据处理类
  	@Component
    @Slf4j
		public class CommonMetaObjectHandler implements MetaObjectHandler {
        @Override
        public void insertFill(MetaObject metaObject) {
            log.info("新建时，开始填充系统字段！");
            this.strictInsertFill(metaObject, "created", LocalDateTime.class, LocalDateTime.now());
            this.strictInsertFill(metaObject, "modified", LocalDateTime.class, LocalDateTime.now());
            this.strictInsertFill(metaObject, "creator", String.class, "TODO 从上下文中获取当前人");
            this.strictInsertFill(metaObject, "operator", String.class, "TODO 从上下文中获取当前人");
            this.strictInsertFill(metaObject, "status", Integer.class, 0);
            this.strictInsertFill(metaObject, "version", Long.class, 1L);
      }

        @Override
        public void updateFill(MetaObject metaObject) {
            log.info("更新时，开始填充系统字段！");
            this.strictUpdateFill(metaObject, "modified", LocalDateTime.class, LocalDateTime.now());
            this.strictUpdateFill(metaObject, "operator", String.class, "TODO 从上下文中获取当前人");
        }
}

	2.为XxxDO配置注解@TableField(fill={value})
    /**
     * 数据库修改的时间
     */
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime created;

    /**
     * 数据修改的时间
     */
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime modified;
    
```



#### 集成CaffeineCache实现本地缓存

###### 集成CaffeineCache的方法

```java
缓存的注解：
  1.@Cachable:缓存数据，一般用在查询方法上，将查询到的数据进行缓存
  2.@CachePut：更新缓存，一般用在更新方法上，将数据从缓存中更新
  3.@CacheEvict：删除缓存
集成方法：
  1.pom.xml cache相关的依赖支持
  2.CacheManager Bean类的创建
  3.使用注解标识我们的方法那些需要缓存
```

###### 1.pom.xml cache相关的依赖支持

```java
<!-- Caffeine Cache支持 -->
  <dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context-support</artifactId>
  <version>5.2.6.RELEASE</version>
  </dependency>
  <dependency>
  <groupId>com.github.ben-manes.caffeine</groupId>
  <artifactId>caffeine</artifactId>
  </dependency>
```

###### 2.CacheManager Bean类的创建

```java
@Configuration
@EnableCaching
@Slf4j
public class CaffeineCacheConfig {

    /**
     * CacheManager实现类
     * @return
     */
    @Bean("cacheManager")
    public CacheManager cacheManager(){
        //创建缓存管理对象
        SimpleCacheManager simpleCacheManager = new SimpleCacheManager();

        //创建缓存的集合
        ArrayList<CaffeineCache> caches = new ArrayList<>();
				//设置缓存的基本配置，缓存名称，缓存容量，缓存存在时间
        caches.add(new CaffeineCache("users-cache",
                Caffeine.newBuilder()
                        .maximumSize(1000)
                        .expireAfterAccess(120, TimeUnit.SECONDS)
                        .build()
                ));
        simpleCacheManager.setCaches(caches);

        return  simpleCacheManager;
    }
}
```

###### 3.在controller层的方法上使用对应的注解，启动缓存

```java
 		/**
     * 新建用户
     * @param userDTO
     * @return
     */
    @CacheEvict(cacheNames = "users-cache", allEntries = true)
    @PutMapping()
    public ResponseResult save(@Validated(InsertValidationGroup.class)@RequestBody UserDTO userDTO){

        int save = userService.save(userDTO);
        if(save == 1){
            return  ResponseResult.success(null, ErrorCodeEnum.SUCCESSS);
        }else {
            return ResponseResult.failure(ErrorCodeEnum.INSERT_FAILURE);
        }
    }

		/**
     * 查询用户信息
     * @param pageNo
     * @param pageSize
     * @param userQueryDTO
     * @return
     */
    @Cacheable(cacheNames = "users-cache")
    @GetMapping
    public ResponseResult<PageResult> query(@NotNull(message = "不能为空")Integer pageNo,
                                            @NotNull(message = "不能为空")Integer pageSize,
                                            @Validated UserQueryDTO userQueryDTO){

        log.info("未使用缓存！");

        PageResult<List<UserDTO>> query = userService.query(new PageQuery<>(pageNo, pageSize, userQueryDTO));

        List<UserVO> collect = Optional.ofNullable(query.getData())
                .map(List::stream)
                .orElseGet(Stream::empty)
                .map(userDTO -> {
                    UserVO userVo = new UserVO();

                    BeanUtils.copyProperties(userDTO, userVo);
                    userVo.setPassword("********");
                    userVo.setPhone(userDTO.getPhone().replace("(\\d{3})\\d{4}(\\d{4}))","$1****$2"));
                    return userVo;
                }).collect(Collectors.toList());

        PageResult<List<UserVO>> pageResult = new PageResult<>();

        BeanUtils.copyProperties(query, pageResult);
        pageResult.setData(collect);

        return ResponseResult.success(pageResult, ErrorCodeEnum.SUCCESSS);
    }

```



#### 集成Lombok注解

```java

```



#### 集成校验框架实现自动/手动分组数据校验

```java
1.在pom.xml中引入相关依赖。
  <!-- validation依赖 -->
  <dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-validation</artifactId>
  </dependency>
2.添加分组校验，根据添加和更新进行分组，创建IntertValidationGruop和UpdateValidationGroup两个接口.
3.在待验证的实体中添加相应的注解。
  
   @NotBlank(message = "用户名不能为空!")
    private String username;

    /**
     * 密码
     */
    @NotBlank(message = "密码不能为空!",groups = {InsertValidationGroup.class})
    @Length(max = 18, min = 6, message = "密码长度不能少于6位，不能多于18位！")
    private String password;

4.在Controller层添加相应的注解。
 	@RestController
@RequestMapping("/api/users")
@Slf4j
@Validated
public class UserController {

    @Autowired
    private UserService userService;

    /**
     * 新建用户
     * @param userDTO
     * @return
     */
    @PutMapping()
    public ResponseResult save(@Validated(InsertValidationGroup.class)@RequestBody UserDTO userDTO){

        int save = userService.save(userDTO);
        if(save == 1){
            return  ResponseResult.success(null, ErrorCodeEnum.SUCCESSS);
        }else {
            return ResponseResult.failure(ErrorCodeEnum.INSERT_FAILURE);
        }


    }
5.做参数检验工具类，完成service层的手动校验。
  public class ValidatorUtils {
      private static Validator validator = Validation.buildDefaultValidatorFactory().getValidator();

      public static <T> void validate(T object, Class... groups){
          Set<ConstraintViolation<T>> validate = validator.validate(object, groups);

          //如果校验结果不为空
          if(!CollectionUtils.isEmpty(validate)){
              StringBuilder stringBuilder = new StringBuilder();
              validate.forEach(tConstraintViolation -> {
                      stringBuilder.append(tConstraintViolation.getMessage());
                  });

              //抛出一个运行时异常
              throw new RuntimeException(stringBuilder.toString());
          }

      }
	}
  //service
  //手动校验功能
  ValidatorUtils.validate(pageQuery);
```

#### 实现统一异常处理

###### 	1.实现一个异常处理的类，并且使用@ControllerAdvice注解修饰

```java
@ControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
}   
  
```

######  	2.在该类中创建捕获运行时异常和全局异常的方法（RuntimeExceptionHandle和ThreableExceptionHandle）

```java


    /**
     * 拦截运行时异常
     * @param e
     */
    @ResponseBody
    @ExceptionHandler(value = RuntimeException.class)
    public ResponseResult runtimeExceptionHandle(RuntimeException e){

        log.error("捕捉到的运行时异常", e.getMessage());

        return  ResponseResult.failure(ErrorCodeEnum.UNKNOWN_ERROR.getCode(), e.getMessage());
    }

    /**
     * 拦截全局异常
     * @param th
     * @return
     */
    @ResponseBody
    @ExceptionHandler(value = Throwable.class)
    public ResponseResult throwableHandle(Throwable th){
        log.error("捕捉Throwable异常：", th);
        return  ResponseResult.failure(ErrorCodeEnum.UNKNOWN_ERROR.getCode(), th.getMessage());
    }
```

###### 	3.创建业务类异常类

```java
public class BusinessException extends RuntimeException{

    /**
     * 异常编号
     */
    @Getter
    private final String code;

    /**
     * 根据枚举类构造业务异常
     * @param errorCodeEnum
     */
    public BusinessException(ErrorCodeEnum errorCodeEnum){
        super(errorCodeEnum.getMessage());
        this.code = errorCodeEnum.getCode();
    }

    /**
     * 自定义消息体构造业务异常
     * @param errorCodeEnum
     * @param message
     */
    public BusinessException(ErrorCodeEnum errorCodeEnum, String message){
        super(message);
        this.code = errorCodeEnum.getCode();
    }

    /**
     * 根据异常来构造业务类异常
     * @param errorCodeEnum
     * @param throwable
     */
    public BusinessException(ErrorCodeEnum errorCodeEnum, Throwable throwable){
        super(throwable.getMessage());
        this.code = errorCodeEnum.getCode();
    }

}
```

###### 	4.在全局异常处理类中捕获业务类异常

```java
/**
     * 捕获业务类异常
     * @param b
     * @return
     */
    @ResponseBody
    @ExceptionHandler(value = BusinessException.class)
    public ResponseResult businessExceptionHandle(BusinessException b){

        log.error("捕获到业务类异常");
        return ResponseResult.failure(b.getCode(), b.getMessage());
    }
```



#### 集成Guava令牌桶实现全局限流器

###### 集成Guava令牌桶的方法步骤

```java
1.先把pom.xml引入Guava工具包的支持
2.定义一个拦截器，实现令牌的发放和获取
3.将拦截器配置到web系统中
```

###### 1.先把pom.xml引入Guava工具包的支持

```java
<!-- Guava 支持 -->
  <dependency>
  	<groupId>com.google.guava</groupId>
  	<artifactId>guava</artifactId>
 	 	<version>28.2-jre</version>
  </dependency>
```

###### 2.定义一个拦截器，实现令牌的发放和获取

```java
@Component
@Slf4j
public class RateLimitInterceptor implements HandlerInterceptor {

    /**
     * 限流器实例（QPS限制为10）
     */
    private static final RateLimiter rateLimiter = RateLimiter.create(1);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        if(!rateLimiter.tryAcquire()){
            log.error("系统被限流了");

            throw new BusinessException(ErrorCodeEnum.RATE_LIMIT_ERROR);
        }

        return true;
    }
}
```

###### 3.将拦截器配置到web系统中

```java
@Configuration
@EnableWebMvc
@Slf4j
public class WebConfig implements WebMvcConfigurer {

    @Resource
    private RateLimitInterceptor rateLimitInterceptor;

    /**
     * 向web中添加拦截器
     * @param registry
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {

        //配置限流拦截器，拦截所有以/api/开头的请求
        registry.addInterceptor(rateLimitInterceptor)
                .addPathPatterns("/api/**");
    }

    /**
     * 静态资源拦截器
     * @param registry
     */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {

    }
}
```



#### 使用线程池实现任务异步处理

```

```



#### 使用TWR实现文件上传和下载

###### 实现流程

```
1.文件上传接口的controller负责处理文件上传
2.文件上传的服务接口，通过接口的形式来定义文件上传的功能
3.实现文件上传的业务逻辑
4.文件下载，采用静态路径映射的方式实现
```

###### 1.文件上传接口的controller负责处理文件上传

```java
@RestController
@RequestMapping("/api/file")
@Slf4j
public class FileController {

    @Resource(name = "LocalFileServiceImpl")
    private FileService fileService;

    @PostMapping("/upload")
    public ResponseResult<String> fileUpload(@NotNull MultipartFile file){
        //文件上传
        try {
            fileService.upload(file.getInputStream(), file.getOriginalFilename());
        } catch (IOException e) {
            log.error("本地上传文件失败！");
            throw new BusinessException(ErrorCodeEnum.FILE_UPLOAD_FAILURE);
        }
        return ResponseResult.success(file.getOriginalFilename(),ErrorCodeEnum.SUCCESSS);

    }
}
```

###### 2.文件上传的服务接口，通过接口的形式来定义文件上传的功能

```java
public interface FileService {

    /**
     * 使用输入流上传文件
     * @param inputStream
     * @param fileName
     */
    void upload(InputStream inputStream, String fileName);

    /**
     * 使用文件直接上传
     * @param file
     */
    void upload(File file);

}
```

###### 3.实现文件上传的业务逻辑

```java
@Service("LocalFileServiceImpl")
@Slf4j
public class LocalFileServiceImpl implements FileService {


    private static final String BUCKET = "uploads";

    @Override
    public void upload(InputStream inputStream, String fileName) {

        //拼接真正的存储位置
        String storagePath = BUCKET + "/" + fileName;


        try (
                //JDK8 TWR 不能关闭外部资源的只能关闭内部资源
                InputStream innerInputStream = inputStream;
                FileOutputStream fileOutputStream = new FileOutputStream(new File(storagePath));
        ) {
            //拷贝缓冲区
            byte[] buffer = new byte[1024];
            //读取文件流的长度
            int len = 0;
            //循环读取inoutStream中数据写入到ouputStream
            while ((len = innerInputStream.read(buffer)) > 0) {
                fileOutputStream.write(buffer, 0, len);
            }

            //冲刷流
            fileOutputStream.flush();


        } catch (Exception e) {
            log.error("本地上传文件失败！");
            throw new BusinessException(ErrorCodeEnum.FILE_UPLOAD_FAILURE);
        }

    }

    @Override
    public void upload(File file) {

        try {
            upload(new FileInputStream(file), file.getName());
        } catch (FileNotFoundException e) {
            throw new BusinessException(ErrorCodeEnum.FILE_UPLOAD_FAILURE);
        }
    }
}
```

###### 4.文件下载，采用静态路径映射的方式实现

```java
 /**
     * 静态资源拦截器
     * @param registry
     */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/uploads/**")
                .addResourceLocations("file:/Users/Emotibot/Desktop/sbtools/uploads/");
    }
```



#### 集成EasyExcel实现Excel异步导出

```
1.pom.xml引入相关依赖
2.
```



#### 实现代码VO/DTO/DO的代码分层

```

```



#### 使用Stream实现集合操作

```

```



#### 集成Swagger2实现接口文档的自动生成

```java
1.引入Swgger2的依赖包
2.配置Swagger2的配置类
3.在Cotroller及相关的实体中写对应的注解
```



#### 使用TraceId实现系统日志追踪

###### 集成步骤

```java
1.建立一个过滤器，在过滤器中给线程设置TraceId.
2.将日志配置文件进行修改，把TraceId打印到日志中.
```

###### 1.建立一个过滤器，在过滤器中给线程设置TraceId.

```java
@WebFilter(urlPatterns = "/*")
@Order(1)
public class TraceIdFilter implements Filter {

    private static  final String TRACE_ID = "traceId";

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {

        String traceId = servletRequest.getParameter(TRACE_ID);
        if(StringUtils.isEmpty(traceId)){
            traceId = UUID.randomUUID().toString();
        }
        MDC.put(TRACE_ID,traceId);
        filterChain.doFilter(servletRequest, servletResponse);
    }
}
```

###### 2.将日志配置文件进行修改，把TraceId打印到日志中.

```yaml
logging:
  pattern:
    console: "%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr([%X{traceId}]) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"


```

