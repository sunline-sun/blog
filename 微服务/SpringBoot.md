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
    //根据exclude进行排除
		checkExcludedClasses(configurations, exclusions);
    //exclusions也排除
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
  - 

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

### 自定义starter（场景启动器）
- 我们可以通过自定义starter加载自己需要的bean
