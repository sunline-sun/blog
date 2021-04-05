### springBoot介绍
- SpringBoot是为了简化spring应用的搭建和开发，减少了spring中大量的配置文件。

### SpringBoot特征
- 可以创建可执行的war包和jar包
- 内嵌tomcat等sevlet容器
- 提供自动配置的starter
- 尽可能自动化配置spring容器
- 不需要xml配置

### 自动装配原理
- https://www.processon.com/view/link/605b4b9de401fd4c03961a84
- @SpringBootApplication 标注这个是springboot的启动类
  - @Target 注解的注解，标注这个注解可以标记在哪
  - @Retention 注解的注解，标注这个注解标注的类编译的时候用什么方法保留
  - @Documented Java doc会生成注解信息
  - @inherited 是否会被继承
  - @SpringBootConfiguration 表示这是一个SpringBoot配置类
  - @Configuration 表明这是个配置类
  - @EnableAutoConfiguration 开启自动配置功能，这是自动装配的主要注解
    - @AutoConfigurationPackage 注册一个bean，将当前配置类所在包保存在这个bean里，供spring内部使用
    - @Import(EnableAutoConfigurationImportSelector.class) 导入配置，EnableAUtoConfigurationImportSelector实现了defferredImportSelector,spring 内部解析import的时候会调用getAutoConfigurationEntry方法
      - getAutoConfigurationEntry会扫描具有spring.properties文件的jar包
<details>
  <summary>源代码</summary>
  
```java
if (!isEnabled(annotationMetadata)) {
			return EMPTY_ENTRY;
		}
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
    // 从spring.properties 中获取到候选的自动配置类
		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
    //排重
		configurations = removeDuplicates(configurations);
    //根据EnableConfiguration注解中的属性，获取不需要自动配置的类名单
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    //校验exclude的类是否在这个属性下
		checkExcludedClasses(configurations, exclusions);
    //exclude进行排除，exclusions也排除
		configurations.removeAll(exclusions);
    //通过读取spring.factories 中的OnBeanCondition\OnClassCondition\OnWebApplicationCondition进行过滤
		configurations = getConfigurationClassFilter().filter(configurations);
    //这个方法是调用实现了AutoConfigurationImportListener  的bean..  分别把候选的配置名单，和排除的配置名单传进去做扩展
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return new AutoConfigurationEntry(configurations, exclusions);
```
</details>
  - @ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) }) 扫描包
    - TypeExcludeFilter：springboot对外提供的扩展类， 可以供我们去按照我们的方式进行排除
    - AutoConfigurationExcludeFilter：排除所有配置类并且是自动配置类中里面的其中一个

### springboot自动配置步骤
- springBoot启动类中，@SpringBootApplication中包含@EnableAutoConfiguration注解，@EnableAutoConfiguration中通过@Import(EnableAutoConfigurationImportSelector.class)导入配置，EnableAutoConfigurationImportSelector中的getAutoConfigurationEntry方法会获取所有需要加载的bean
  - 获取候选的自动配置类
    - 获取spring.property文件中的所有配置，把这些配置存到map里
    - 从map里获取enableAutoConfiguration下的配置类
  - 通过linkedhashset排重，把重复的配置类去重
  - 根据enableConfiguration注解中的exclude和exclusions属性，去掉对应的配置类
  - 通过从spring.properties中读取到的AutoConfigurationImportFilter也就是OnClassCondition这些，对候选的配置类进行过滤
  - 调用实现了AutoconfigurationImportListener的bean，吧候选的配置类和排除的配置类传进去做扩展

### @Conditional派生注解（Spring注解版原生的@Conditional作用）
- 作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效；
- @Conditional扩展注解作用（判断是否满足当前指定条件）
- @ConditionalOnJava系统的java版本是否符合要求
- @ConditionalOnBean容器中存在指定Bean；
- @ConditionalOnMissingBean容器中不存在指定Bean；
- @ConditionalOnExpression 满足SpEL表达式指定
- @ConditionalOnClass系统中有指定的类
- @ConditionalOnMissingClass系统中没有指定的类
- @ConditionalOnSingleCandidate 容器中只有一个指定的Bean，或者这个Bean是首选Bean
- @ConditionalOnProperty系统中指定的属性是否有指定的值
- @ConditionalOnResource 类路径下是否存在指定资源文件
@ConditionalOnWebApplication当前是web环境
@ConditionalOnNotWebApplication 当前不是web环境
@ConditionalOnJndi JNDI存在指定项

### spring-boot-starter作用和自定义starter（场景启动器）的作用
- spring-boot-starter是一个配置了很多jar包的空的maven项目，他的作用是可以让开发人员不用关注需要依赖哪些库，想用某个模块的时候直接依赖这个模块的starter就可以了，starter里会把需要的所有jar包都依赖进来，比如我们需要在项目里使用redis，直接依赖spring-boot-starter-redis就可以了，这个下面会自己依赖需要的库，加载需要的bean到容器里
- 自定义starter也是一样的目的，就是可以把独立于业务之外的模块封装成一个个的starter，需要的时候直接依赖这个starter就可以了，springboot就可以帮我们完成自动装配

### 自定义starter步骤
- 创建spring.properties配置文件，定义需要加载的配置类
- 定义配置类，创建需要实例化的bean
- 引入这个starter，就可以实现自动配置了

### starter命名规范
- 官方命名：spring-boot-starter-*
- 自定义starter：test-spring-boot-starter

### springBoot是怎么通过jar包就可以启动的
- 首先依赖的spring-boot-maven-plugin包，会帮我们做两件事
  - 生成manifest.mf文件
    - manifest.mf中的main-class是jarlauncher，当运行的时候也就是执行这个来加载类，这个里有自定义的类加载器，用来实现加载jar中的jar包（嵌套的jar），因为默认是不支持这种的
    - start-class是我们本地的启动类，在加载完类之后会通过反射的方式调用这个本地的启动类
  - 把依赖的jar包也打进去了
- 当运行jar包的时候，会先找到mainfest.mf文件里的main-class，然后通过加载BOOT-INF/classes和 BOOT-INF/lib目录下的jar文件，实现fat jar的启动
- springboot通过扩展jarfile、jarURLconnection和URLStreamHandler，实现jar in jar 的资源的加载
- springboot通过扩展URLClassLoader-LauncharURLClassLoader，实现了jar in jar class文件的加载
- 类加载完后，会找到manifest.mf中的start-class，反射的方式调用这个自己定义的启动类
