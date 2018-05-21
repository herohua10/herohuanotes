###  Spring

#### Spring框架的四大原则

1. 使用POJO进行轻量级和最小侵入式开发。
2. 通过依赖注入和基于接口编程实现松耦合。
3. 通过AOP和默认习惯进行声明式编程。
4. 使用AOP和模板（template）减少模块化代码。

#### 控制反转和依赖注入

​	控制反转是通过依赖注入实现的。所谓依赖注入指的是容器负责创建对象和维护对象之间的的依赖关系，而不是对象本身负责自己的创建和解决自己的依赖。

​	依赖注入的主要目的是为了解耦，体现了一种“组合”的理念。而通过继承的方式会使子类与父类耦合，而组合另一个类则使耦合度大大降低。

#### AnnotationConfigApplicationContext是什么？

​	作为Spring容器，接受一个配置类作为输入参数。

```java
AnnotationConfigApplicationContext context = 
    					new AnnotationConfigApplicationContext(DiConfig.class);
```

#### 何时使用Java配置和注解配置？

​	使用原则：全局使用Java配置（如数据库相关配置、MVC相关配置），业务Bean的配置使用注解配置（@Service、@Component、@Repository、@Controller）。

​	java配置是通过@Configuration和@Bean实现的。

​	@Configuration声明当前类是一个配置类，相当于Spring配置的xml文件。

​	@Bean注解在方法上，声明当前方法的返回值是一个Bean。

#### Bean的Scope

​	Scope描述的是Spring容器如何新建Bean的实例的。使用注解@Scope("")配置。

1. Singleton：一个Spring容器中只有一个Bean的实例，此为Spring的默认配置，全容器共享一个实例。
2. Prototype：每次调用新建一个Bean的实例。
3. Request：Web项目中，给每一个http request新建一个Bean实例。
4. Session：Web项目中，给每一个http session新建一个Bean实例。
	. GlobalSession：这个只在portal应用中游泳，给每一个global http session新建一个Bean实例。	

#### Spring的El表达式和资源调用

使用Spring的El表达式语言实现资源的注入。

在@Value的参数中使用表达式。

```java
@Configuration
@ComponentScan("com.wisely.highlight_spring4.ch2.el")
@PropertySource("classpath:com/wisely/highlight_spring4/ch2/el/test.properties")
public class ElConfig {
	//注入普通字符串
	@Value("I love u!")
	private String normal;
	//注入操作系统属性
	@Value("#{systemProperties['os.name']}")
	private String osName;
	//注入表达式结果
	@Value("#{T(java.lang.Math).random()*100.0}")
	private double randomNumber;
	//注入其他Bean属性
	@Value("#{demoService.another}")
	private String formAnother;
	//注入文件资源
	@Value("classpath:com/wisely/highlight_spring4/ch2/el/test.txt")
	private Resource testFile;
	//注入网址资源
	@Value("http://www.baidu.com")
	private Resource testUrl;
	//注入配置文件的属性，注意是$
	@Value("${book.name}")
	private String bookName;
	@Autowired
	private Environment environment;
	@Bean
	public static PropertySourcesPlaceholderConfigurer propertyConfigure() {
		return new PropertySourcesPlaceholderConfigurer();
	}
	public void outputResource() {
		try {
			System.out.println(normal);
			System.out.println(osName);
			System.out.println(randomNumber);
			System.out.println(formAnother);
			System.out.println(IOUtils.toString(testFile.getInputStream()));
			System.out.println(IOUtils.toString(testUrl.getInputStream()));
			System.out.println(bookName);
			System.out.println(environment.getProperty("book.author"));
		}catch(Exception e) {
			e.printStackTrace();
		}
	}
}
```

#### 如何在将SpringBoot项目打包、发布

1. 在子模块pom文件中配置Maven打包插件，让jar包提供可执行的服务。

   ```xml
   <build>
     	<plugins>
     		<!-- springboot打包插件 -->
     		<plugin>
     			<groupId>org.springframework.boot</groupId>
     			<artifactId>spring-boot-maven-plugin</artifactId>
     			<version>1.3.3.RELEASE</version>
     			<executions>
     				<execution>
     					<goals>
     						<goal>repackage</goal>
     					</goals>
     				</execution>
     			</executions>
     		</plugin>
     	</plugins>
     	<finalName>demo</finalName>		<!-- jar包名 -->
     </build>
   ```

2. 在父项目上Run as Maven build命令，Goals填clean package。

3. 生成的可执行的jar包在target目录下。

4. 部署：在命令行中，进入到jar所在的文件夹中，输入java -jar demo.jar命令，项目将会部署。相当于在eclipse运行启动类。jar包中内嵌了tomcat。此时便可以访问SpringBoot服务了。

#### MockMvc测试用例

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserControllerTest {
	//注入web环境
	@Autowired
	private WebApplicationContext wac;
	
	private MockMvc mockMvc;
	//每次执行@Test方法前执行
	@Before
	public void setup() {
		mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
	}
	//模拟一个get请求
	@Test
	public void whenQuerySuccess() throws Exception {
		mockMvc.perform(MockMvcRequestBuilders.get("/")
						.contentType(MediaType.APPLICATION_JSON_UTF8))
            			 .param("username", "huajian")	
			   .andExpect(MockMvcResultMatchers.status().isOk())
			   .andExpect(MockMvcResultMatchers.jsonPath("$.length()").value(3));
	}
}
```

#### @RequestParam注解和@RequestParam区别

都是映射请求参数到java方法的请求参数。

##### @RequestParam

用来处理Content-Type: 为 application/x-www-form-urlencoded编码的内容。（Http协议中，如果不指定Content-Type，则默认传递的参数就是application/x-www-form-urlencoded类型）

RequestParam可以接受简单类型的属性，也可以接受对象类型。 
实质是将Request.getParameter() 中的Key-Value参数Map利用Spring的转化机制ConversionService配置，转化成参数接收对象或字段。

三个参数：

value：定义参数的名称 

required：定义参数是否是必须的，默认是true

defaultValue：定义参数的默认值

tip：

在`Content-Type: application/x-www-form-urlencoded`的请求中， get 方式中queryString的值，和post方式中 body data的值都会被Servlet接受到并转化到Request.getParameter()参数集中，所以@RequestParam可以获取的到。

##### @RequestBody

处理HttpEntity(请求体)传递过来的数据，一般用来处理非`Content-Type: application/x-www-form-urlencoded`编码格式的数据。

- GET请求中，因为没有HttpEntity，所以@RequestBody并不适用。
- POST请求中，通过HttpEntity传递的参数，必须要在请求头中声明数据的类型`Content-Type`，SpringMVC通过使用HandlerAdapter 配置的HttpMessageConverters来解析HttpEntity中的数据，然后绑定到相应的bean上。

##### 总结

- 在GET请求中，不能使用@RequestBody。
- 在POST请求，可以使用@RequestBody和@RequestParam，但是如果使用@RequestBody，对于参数转化的配置必须统一。

举个例子，在SpringMVC配置了HttpMessageConverters处理栈中，指定json转化的格式，如Date转成‘yyyy-MM-dd’,则参数接收对象包含的字段如果是Date类型，就只能让客户端传递年月日的格式，不能传时分秒。因为不同的接口，它的参数可能对时间参数有不同的格式要求，所以这样做会让客户端调用同事对参数的格式有点困惑，所以说扩展性不高。

如果使用@RequestParam来接受参数，可以在接受参数的model中设置@DateFormat指定所需要接受时间参数的格式。

另外，使用@RequestBody接受的参数是不会被Servlet转化统一放在request对象的Param参数集中，@RequestParam是可以的。

综上所述，一般情况下，推荐使用@RequestParam注解来接受Http请求参数。

#### JsonPath用法

##### 操作符

| Operator                  | Description                                                  |
| ------------------------- | ------------------------------------------------------------ |
| `$`                       | The root element to query. This starts all path expressions. |
| `@`                       | The current node being processed by a filter predicate.      |
| `*`                       | Wildcard. Available anywhere a name or numeric are required. |
| `..`                      | Deep scan. Available anywhere a name is required.            |
| `.<name>`                 | Dot-notated child                                            |
| `['<name>' (, '<name>')]` | Bracket-notated child or children                            |
| `[<number> (, <number>)]` | Array index or indexes                                       |
| `[start:end]`             | Array slice operator                                         |
| `[?(<expression>)]`       | Filter expression. Expression must evaluate to a boolean value. |

##### 函数

| Function | Description                                                  | Output  |
| -------- | ------------------------------------------------------------ | ------- |
| min()    | Provides the min value of an array of numbers                | Double  |
| max()    | Provides the max value of an array of numbers                | Double  |
| avg()    | Provides the average value of an array of numbers            | Double  |
| stddev() | Provides the standard deviation value of an array of numbers | Double  |
| length() | Provides the length of an array                              | Integer |

##### 过滤操作符

| Operator | Description                                                  |
| -------- | ------------------------------------------------------------ |
| ==       | left is equal to right (note that 1 is not equal to '1')     |
| !=       | left is not equal to right                                   |
| <        | left is less than right                                      |
| <=       | left is less or equal to right                               |
| >        | left is greater than right                                   |
| >=       | left is greater than or equal to right                       |
| =~       | left matches regular expression [?(@.name =~ /foo.*?/i)]     |
| in       | left exists in right [?(@.size in ['S', 'M'])]               |
| nin      | left does not exists in right                                |
| subsetof | left is a subset of right [?(@.sizes subsetof ['S', 'M', 'L'])] |
| size     | size of left (array or string) should match right            |
| empty    | left (array or string) should be empty                       |

##### 例子

```json
{
    "store": {
        "book": [
            {
                "category": "reference",
                "author": "Nigel Rees",
                "title": "Sayings of the Century",
                "price": 8.95
            },
            {
                "category": "fiction",
                "author": "Evelyn Waugh",
                "title": "Sword of Honour",
                "price": 12.99
            },
            {
                "category": "fiction",
                "author": "Herman Melville",
                "title": "Moby Dick",
                "isbn": "0-553-21311-3",
                "price": 8.99
            },
            {
                "category": "fiction",
                "author": "J. R. R. Tolkien",
                "title": "The Lord of the Rings",
                "isbn": "0-395-19395-8",
                "price": 22.99
            }
        ],
        "bicycle": {
            "color": "red",
            "price": 19.95
        }
    },
    "expensive": 10
}
```

![](C:\Users\华健\Pictures\jsonpath.PNG)

#### @JsonView使用步骤

@JsonView注解用来过滤序列化对象的字段属性，简单来说就是定义一个标签，根据controller的JsonView属性，将实体类中不同标签的属性进行分类显示。

1. 使用接口来声明多个视图；
2. 在值对象的get方法上指定视图；
3. 在Controller方法上指定视图。

```Java
public class User {
	//1.使用接口来声明多个视图
	public interface UserSimpleView {};
	public interface UserDetailView extends UserSimpleView {};
	
	private String username;
	private String password;
    //2.在值对象的get方法上指定视图；
	@JsonView(UserSimpleView.class)
	public String getUsername() {
		return username;
	}
	public void setUsername(String username) {
		this.username = username;
	}
	//2.在值对象的get方法上指定视图；
	@JsonView(UserDetailView.class)
	public String getPassword() {
		return password;
	}
	public void setPassword(String password) {
		this.password = password;
	}	
}
//3.在Controller方法上指定视图。
@JsonView(User.UserSimpleView.class)
@RequestMapping(value = "/user", method=RequestMethod.GET)
public List<User> query(@RequestParam String username){

    System.out.println("username:" + username);

    List<User> users = new ArrayList<User>();
    users.add(new User());
    users.add(new User());
    users.add(new User());
    return users;
}
```

#### 日期类型参数的处理

前后台只传递时间戳，前台根据不同场景将时间戳转换成所需格式的日期格式。

#### 使用@Valid进行参数校验

例子：

```java
//1.在实体类字段上添加校验规则
public class User {
    //密码不为空
    @NotBlank
	private String password;
    //....
}
//2.在Controller方法中使用@Valid开启校验
@PostMapping
public User create(@Valid @RequestBody User user, BindingResult errors) {
	
    if(errors.hasErrors()) {
        errors.getAllErrors().stream().forEach(error -> { 
           FieldError fieldError = (FieldError)error;
           System.out.println(fieldError.getField()+":"+fieldError.getDefaultMessage());
        });
	}

    System.out.println(ReflectionToStringBuilder.toString(user, 		     ToStringStyle.MULTI_LINE_STYLE));

    user.setId("1");
    return user;
}
//备注：这里一个@Valid的参数后必须紧挨着一个BindingResult 参数，否则spring会在校验不通过时直接抛出异常
```

常用注解

| **验证注解**                                 | **验证的数据类型**                                           | **说明**                                                     |
| -------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| @AssertFalse                                 | Boolean,boolean                                              | 验证注解的元素值是false                                      |
| @AssertTrue                                  | Boolean,boolean                                              | 验证注解的元素值是true                                       |
| @NotNull                                     | 任意类型                                                     | 验证注解的元素值不是null                                     |
| @Null                                        | 任意类型                                                     | 验证注解的元素值是null                                       |
| @Min(value=值)                               | BigDecimal，BigInteger, byte,short, int, long，等任何Number或CharSequence（存储的是数字）子类型 | 验证注解的元素值大于等于@Min指定的value值                    |
| @Max（value=值）                             | 和@Min要求一样                                               | 验证注解的元素值小于等于@Max指定的value值                    |
| @DecimalMin(value=值)                        | 和@Min要求一样                                               | 验证注解的元素值大于等于@ DecimalMin指定的value值            |
| @DecimalMax(value=值)                        | 和@Min要求一样                                               | 验证注解的元素值小于等于@ DecimalMax指定的value值            |
| @Digits(integer=整数位数, fraction=小数位数) | 和@Min要求一样                                               | 验证注解的元素值的整数位数和小数位数上限                     |
| @Size(min=下限, max=上限)                    | 字符串、Collection、Map、数组等                              | 验证注解的元素值的在min和max（包含）指定区间之内，如字符长度、集合大小 |
| @Past                                        | java.util.Date,java.util.Calendar;Joda Time类库的日期类型    | 验证注解的元素值（日期类型）比当前时间早                     |
| @Future                                      | 与@Past要求一样                                              | 验证注解的元素值（日期类型）比当前时间晚                     |
| @NotBlank                                    | CharSequence子类型                                           | 验证注解的元素值不为空（不为null、去除首位空格后长度为0），不同于@NotEmpty，@NotBlank只应用于字符串且在比较时会去除字符串的首位空格 |
| @Length(min=下限, max=上限)                  | CharSequence子类型                                           | 验证注解的元素值长度在min和max区间内                         |
| @NotEmpty                                    | CharSequence子类型、Collection、Map、数组                    | 验证注解的元素值不为null且不为空（字符串长度不为0、集合大小不为0） |
| @Range(min=最小值, max=最大值)               | BigDecimal,BigInteger,CharSequence, byte, short, int, long等原子类型和包装类型 | 验证注解的元素值在最小值和最大值之间                         |
| @Email(regexp=正则表达式,flag=标志的模式)    | CharSequence子类型（如String）                               | 验证注解的元素值是Email，也可以通过regexp和flag指定自定义的email格式 |
| @Pattern(regexp=正则表达式,flag=标志的模式)  | String，任何CharSequence的子类型                             | 验证注解的元素值与指定的正则表达式匹配                       |
| @Valid                                       | 任何非原子类型                                               | 指定递归验证关联的对象；如用户对象中有个地址对象属性，如果想在验证用户对象时一起验证地址对象的话，在地址对象上加@Valid注解即可级联验证 |

#### 什么是RESTful API？

1. 使用URL描述资源。
2. 使用Http方法描述行为。
3. 使用Http状态码来表示不同结果。
4. 使用json交互数据。
5. RESTful只是一种风格，并不是一种强制的标准。

#### 使用Swagger自动生成文档

1. 引入两个依赖

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.8.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.8.0</version>
</dependency>
```

2. 在启动类上加上注解@EnableSwagger2
3. 访问http://localhost:8060/swagger-ui.html
4. 使用常用的三个注解描述方法和参数：@ApiOperation、@ApiModelProperty、@ApiParam。具体看文档。


####  使用WireMock伪造Restful服务

1. 简单介绍

   WireMock 是基于 HTTP 的模拟器。它具备 HTTP 响应存根、请求验证、代理/拦截、记录和回放功能。

当开发人员的开发进度不一致时，可以依赖 WireMock 构建的接口，模拟不同请求与响应，从而避某一模块的开发进度。

2. 下载文件

   自行下载启动 WireMock 服务的 jar 包。

3. 启动服务

		在 jar 包所在目录执行如下命令：

		java -jar wiremock-standalone-2.13.0.jar --port 8062（指定端口）

4. 添加依赖

```xml
<dependency>
    <groupId>com.github.tomakehurst</groupId>
    <artifactId>wiremock</artifactId>
</dependency>
```

5. 编写简单的响应

```java
import static com.github.tomakehurst.wiremock.client.WireMock.aResponse;
import static com.github.tomakehurst.wiremock.client.WireMock.configureFor;
import static com.github.tomakehurst.wiremock.client.WireMock.get;
import static com.github.tomakehurst.wiremock.client.WireMock.removeAllMappings;
import static com.github.tomakehurst.wiremock.client.WireMock.stubFor;
import static com.github.tomakehurst.wiremock.client.WireMock.urlPathEqualTo;

public class MockServer {
	public static void main(String[] args) {	
		//连接WireMock服务器端口
		configureFor(8062);
		//删除旧的规则
		removeAllMappings();
		//伪造请求
		stubFor(get(urlPathEqualTo("user/1"))
				.willReturn(aResponse()
					.withBody("{\"id\":\"1\"}")
					.withStatus(200)));
	}
}
//get()：get 请求
//withStatus()：返回状态
//withBody()：返回数据
```

6. 在浏览器中输入请求 http://localhost:8062/user/1 

   得到预期返回数据。

7. 优化

   ​	在实际业务代码中，调用某个接口不可能只返回简单的 JSON 字符串。如果将这些字符串写到上文的代码中，当需要修改数据结构时将是一个灾难，因此我们可将返回的数据写入到文件进行维护。


```Java
import java.io.IOException;
import org.apache.commons.io.FileUtils;
import org.apache.commons.lang.StringUtils;
import org.springframework.core.io.ClassPathResource;

public class MockServer {

	public static void main(String[] args) throws IOException {	
		//连接WireMock服务器端口
		configureFor(8062);
		//删除旧的规则
		removeAllMappings();
		
		mock("/user/1", "01");
		mock("/user/2", "02");
	}
	
	public static void mock(String url, String file) throws IOException {
		//从文件中去除响应内容
		ClassPathResource resource = 
            			new ClassPathResource("mock/response/"+file+ ".txt");
		String content = StringUtils.join(
            FileUtils.readLines(resource.getFile(), "UTF-8").toArray(), "\n");
		
         stubFor(get(urlPathEqualTo(url))
				.willReturn(aResponse()
					.withBody(content)
					.withStatus(200)));
	}
}
```

#### Spring声明式事务配置

​	事务主要用于处理操作量大，复杂度高的数据。比如说，在人员管理系统中，你删除一个人员，你即需要删除人员的基本资料，也要删除和该人员相关的信息，如信箱，文章等等，这样，这些数据库操作语句就构成一个事务！

​	声明式事务管理建立在**AOP**之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明(或通过基于@Transactional注解的方式)，便可以将事务规则应用到业务逻辑中。

​	 显然声明式事务管理要优于编程式事务管理，这正是spring倡导的非侵入式的开发方式。声明式事务管理使业务代码不受污染，一个普通的POJO对象，只要加上注解就可以获得完全的事务支持。和编程式事务相比，声明式事务唯一不足地方是，后者的最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。但是即便有这样的需求，也存在很多变通的方法，比如，可以将需要进行事务管理的代码块独立为方法等等。

1. spring事务回滚规则

   ​	指示spring事务管理器回滚一个事务的推荐方法是在当前事务的上下文内抛出异常。spring事务管理器会捕捉任何未处理的异常，然后依据规则决定是否回滚抛出异常的事务。

   ​        默认配置下，spring只有在抛出的异常为**运行时unchecked异常**时才回滚该事务，也就是抛出的异常为**RuntimeException的子类(**Errors也会导致事务回滚)，而抛出checked异常则不会导致事务回滚。可以明确的配置在抛出那些异常时回滚事务，包括checked异常。也可以明确定义那些异常抛出时不回滚事务。还可以编程性的通过**setRollbackOnly()**方法来指示一个事务必须回滚，在调用完setRollbackOnly()后你所能执行的唯一操作就是回滚。

2. 配置

   ```xml
   <!-- 配置spring的PlatformTransactionManager，名字为默认值 -->  
   <bean id="transactionManager" 
         			class="org.springframework.jdbc.datasource.DataSourceTransactionManager">  
       <property name="dataSource" ref="dataSource" />  
   </bean>  
   <!-- 开启事务控制的注解支持 -->  
   <tx:annotation-driven transaction-manager="transactionManager"/>
   ```

3. 使用方式1：基于@Transactional注解，显然基于注解的方式更简单易用，更清爽。

   ```java
   @Transactional  
   @Override  
   public void insert(Test test) {  
       dao.insert(test);  
       throw new RuntimeException("test");//抛出unchecked异常，触发事物，回滚  
   } 
   ```
   @Transactional注解属性

   ![@Transactional注解属性](C:\Users\华健\Desktop\截图\@Transactional注解属性.PNG)

4. 使用方式2：基于tx和aop名字空间的xml配置文件

   ```xml
   <!--事务切面配置 -->  
   <tx:advice id="advice" transaction-manager="transactionManager">  
       <tx:attributes>  
           <tx:method name="update*" propagation="REQUIRED" read-only="false" 
                      rollback-for="java.lang.Exception"/>  
           <tx:method name="insert" propagation="REQUIRED" read-only="false"/>  
       </tx:attributes>  
   </tx:advice>  
      
   <aop:config>  
       <aop:pointcut id="testService" 
                     expression="execution (* com.baobao.service.MyBatisService.*(..))"/>  
       <aop:advisor advice-ref="advice" pointcut-ref="testService"/>  
   </aop:config> 
   ```

5. 事务的四个特征

   - **原子性：**一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
   - **一致性：**在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。
   - **隔离性：**数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。
   - **持久性：**事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

6. 事务隔离性问题：

   - **脏读：**脏读就是指当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。
   - **不可重复读：**是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两 次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的，因此称为是不 可重复读。
   - **幻读：**是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。 同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象 发生了幻觉一样。

7. Spring事务的5个隔离级别

		隔离级别是指若干个并发的事务之间的隔离程度。TransactionDefinition 接口中定义了五个表示隔离级别的常量：

- **TransactionDefinition.ISOLATION_DEFAULT**：这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是TransactionDefinition.ISOLATION_READ_COMMITTED。
- **TransactionDefinition.ISOLATION_READ_UNCOMMITTED（读未提交）**：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读，不可重复读和幻读，因此很少使用该隔离级别。比如PostgreSQL实际上并没有此级别。
- **TransactionDefinition.ISOLATION_READ_COMMITTED（读已提交）**：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。
- **TransactionDefinition.ISOLATION_REPEATABLE_READ（可重复读）**：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。该级别可以防止脏读和不可重复读。
- **TransactionDefinition.ISOLATION_SERIALIZABLE（串行化）**：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

8. Spring事务的7个传播行为

    事务传播行为用来描述由某一个事务传播行为修饰的方法被嵌套进另一个方法的时事务如何传播。

   ```java
   public void methodA(){
       methodB();
       //doSomething
    }
    
    @Transaction(Propagation=XXX)
    public void methodB(){
       //doSomething
    }
   ```

   代码中`methodA()`方法嵌套调用了`methodB()`方法，`methodB()`的事务传播行为由`@Transaction(Propagation=XXX)`设置决定。这里需要注意的是`methodA()`并没有开启事务，某一个事务传播行为修饰的方法并不是必须要在开启事务的外围方法中调用。

   ![](C:\Users\华健\Desktop\截图\事务的传播行为.PNG)

9. 事务超时：

   ​	所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 TransactionDefinition 中以 int 的值来表示超时时间，其单位是秒。

     	默认设置为底层事务系统的超时值，如果底层数据库事务系统没有设置超时值，那么就是none，没有超时限制。

10. 事务只读属性

    ​	只读事务用于客户代码只读但不修改数据的情形，只读事务用于特定情景下的优化，比如使用Hibernate的时候。默认为读写事务。

### Maven

#### MAVEN常用命令

- **mvn -v**:查看Maven版本
 - **mvn compile**:编译项目的源代码
 - **mvn test**:使用合适的单元测试框架运行测试，这些测试应该不需要代码被打包或发布。
 - **mvn package**:将编译好的代码打包成可分发的格式，如JAR，WAR，或者EAR
 - **mvn clean**:清理上一次构建的文件（target目录，compile后在项目根目录生成target目录）。
 - **mvn install**:安装包至本地仓库，以备本地的其它项目作为依赖使用。
 - **mvn deploy**:将最终的包复制到远程的仓库，以让其它开发人员与项目共享。

#### 创建目录的两种方式

1. mvn archetype:generate 按照提示进行选择

2. mvn archetype:generate -DgroupId=项目名

   ​					-DartifactId=项目名-模块名

   ​					-Dversion=版本号

   ​					-Dpackage=代码所存在的包名

#### MAVEN生命周期

完整的项目构建过程包括：

清理、编译、测试、打包、集成测试、验证、部署

Maven三套相对独立的生命周期：

1. clean	清理项目

	 pre-clean		执行清理前的工作


   - clean		清理上一次构建生成的所有文件


   - post-clean	执行清理后的文件

2. default（最核心）     构建项目

   包括阶段：compile、test、package、install

3. site           生成项目站点

   - pre-site		在生成项目站点前要完成的工作过
   - site                   生成项目的站点文档
   - post-site           在生成项目站点后要完成的工作
   - site-deploy       发布生成的站点到服务器上

#### Maven依赖范围、传递、排除、冲突

- 依赖范围

项目如果要使用某个框架或依赖，需要把相关jar包引用到classpath中，maven项目提供了三个classpath：编译、测试、运行。

1. compile：默认值。 他表示被依赖项目需要参与当前项目的编译，还有后续的测试，运行周期也参与其中，是一个比较强的依赖。打包的时候通常需要包含进去。
2. test：依赖项目仅仅参与测试相关的工作，包括测试代码的编译和执行，不会被打包，例如：junit。
3. runtime：表示被依赖项目无需参与项目的编译，不过后期的测试和运行周期需要其参与。与compile相比，跳过了编译而已。例如JDBC驱动，适用运行和测试阶段。
4. providede：打包的时候可以不用包进去，别的设施会提供。事实上该依赖理论上可以参与编译，测试，运行等周期。相当于compile，但是打包阶段做了exclude操作。
5. systemed：从参与度来说，和provided相同，不过被依赖项不会从maven仓库下载，而是从本地文件系统拿。需要添加systemPath的属性来定义路径。
6. import：导入的范围，它只是用在dependencyManagement中，表示从其它的pom中导入dependency的配置。

- 依赖传递

  比如：A项目依赖B，B项目依赖C，此时C项目就会传递到A项目中。

- 依赖排除

  如果此时，A项目依赖B项目，无需依赖C项目，那么需要把自动传递的依赖排除掉，通过`<exclusions>`标签实现。 

- 依赖冲突

  两个原则：

  1. 短路优先原则：优先引用路径短的依赖

  2. 路径相同，先声明优先原则

    aven入门教程]: https://www.cnblogs.com/jingmoxukong/p/5591368.html#%E9%85%8D%E7%BD%AE%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F	"maven入门教程"
    aven依赖详解]: https://blog.csdn.net/javaloveiphone/article/details/52081464	"Maven依赖详解"


#### Maven环境隔离

1. 在<build>节点下增加<resources>节点：指定哪种环境需要引入配置文件的目录

   ```xml
   <resources>
       <resource>	<!--具体环境的配置文件目录-->
           <directory>src/main/resource.${deploy.type}}</directory>
           <excludes>
               <exclude>*.jsp</exclude>
           </excludes>
       </resource>
       <resource>	<!--公共配置文件目录-->
           <directory>src/main/resource</directory>
       </resource>
   </resources>
   ```

2. 增加<profiles>节点：添加各种环境

   ```xml
   <profiles>
       <profile>	<!--开发环境-->
           <id>dev</id>
           <activation>
               <activeByDefault>true</activeByDefault>
           </activation>
           <properties>
               <deploy.type>dev</deploy.type>
           </properties>
       </profile>
       <profile>	<!--测试环境-->
           <id>beta</id>
           <properties>
               <deploy.type>beta</deploy.type>
           </properties>
       </profile>
       <profile>	<!--线上环境-->
           <id>prod</id>
           <properties>
               <deploy.type>prod</deploy.type>
           </properties>
       </profile>
   </profiles>
   ```

3. 修改src/main下目录结构，如下图所示：
   ![Maven环境隔离](C:\Users\华健\Desktop\截图\Maven Profiles.PNG)

4. Maven环境隔离打包命令

   ```bash
   mvn clean package -Dmaven.test.skip=true -Pdev		#开发环境打包
   mvn clean package -Dmaven.test.skip=true -Pbeta		#测试环境打包
   mvn clean package -Dmaven.test.skip=true -Pprod		#线上环境打包
   ```

### Git

#### Git配置用户名及Email

```bash
#用户名(huajian)
git config --global user.name "Your Name"	
#邮箱(hero514346219@163.com)
git config --global user.email "Your Email"
#--global表示你这台机器上所有的Git仓库都会使用这个配置
```

[Git教程]: https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000	"Git教程"

#### Git常用命令

```bash
#初始化一个仓库
git init
#添加文件到git仓库,风两部
#第一步，使用命令git add <file>，注意，可反复多次使用，添加多个文件；实际上就是把要提交的所有修 改放到暂存区（Stage）。
git add <file>
#第二步，使用命令git commit，完成,-m(可选)表示本次commit的说明。一次性把暂存区的所有修改提交到分支。
git commit -m "Your remark"
#随时掌握工作区的状态
git status
#查看文件修改内容
git diff <file>
#HEAD指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令
git reset --hard commit_id
#穿梭前，用git log可以查看提交历史，以便确定要回退到哪个版本。
git log <--pretty=oneline>
#要重返未来，用git reflog查看命令历史，以便确定要回到未来的哪个版本
git reflog
#撤销修改
#场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。
git checkout -- file
#场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令git reset HEAD file，就回到了场景1，第二步按场景1操作。
git reset HEAD file
#场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退，不过前提是没有推送到远程库。
#命令git rm用于删除一个文件。如果一个文件已经被提交到版本库，那么你永远不用担心误删，但是要小心，你只能恢复文件到最新版本，你会丢失最近一次提交后你修改的内容。
git rm file
```

#### Git工作区、版本库、暂存区关系

![git版本库](C:\Users\华健\Pictures\0.jpg)

#### Git关联远程库

```bash
#要关联一个远程库,使用命令
git remote add origin git@server-name:path/repo-name.git
#如关联我的github远程库
git remote add origin git@github.com:herohua10/GitRepo.git
#关联后，使用命令git push -u origin master第一次推送master分支的所有内容,远程库的名字就是origin，这是Git默认的叫法
git push -u origin master
#此后，每次本地提交后，只要有必要，就可以使用命令git push origin master推送最新修改
git push origin master
#如果远程主机的版本比本地版本更新，推送时 Git 会报错，要求先在本地做 git pull 合并差异，然后再推送到远程主机。这时，如果你一定要推送，可以使用 --force 选项（慎用）。
git push -u -f origin master
```

#### Git从远程库克隆

```bash
#要克隆一个仓库，首先必须知道仓库的地址，然后使用git clone命令克隆。
git clone git@github.com:herohua10/gitslills.git
#Git支持多种协议，包括https，但通过ssh支持的原生git协议速度最快。
```

#### Git分支

```bash
#Git鼓励大量使用分支：
#查看分支：
git branch
#创建分支：
git branch <name>
#切换分支：
git checkout <name>
#创建+切换分支：
git checkout -b <name>
#合并某分支到当前分支：
git merge <name>
#删除分支：
git branch -d <name>
#丢弃一个没有被合并过的分支，强行删除。
git branch -D <name>
#删除远程分支
git push origin :<name>
#从远程库clone后只能看到本地的master分支，创建远程的dev分支到本地
git checkout -b dev origin/dev
#指定本地的dev分支与远程origin/dev分支的连接
git branch --set-upstream-to=origin/dev dev
#1.git push -u origin dev 相当于
#2.git branch --set-upstream-to=origin/dev dev + git push origin dev
#如果远程仓库没有dev分支，用1，相当于将dev分支push到远程(在远程库建立dev分支，分支与分支之间的流通道自动建立)
#如果远程仓库已有分支dev，可以用2，先建立本地dev与远程dev分支之间的流通道，然后推送
```

![切换到dev分支,并commit了一次](C:\Users\华健\Pictures\1.png)

![切换到master分支，未merge](C:\Users\华健\Pictures\2.png)

#### Git分支合并冲突

当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

用`git log --graph`命令可以看到分支合并图。

![出现冲突](C:\Users\华健\Pictures\11.png)

![解决冲突合并](C:\Users\华健\Pictures\111.png)

#### Git分支策略

通常，合并分支时，如果可能，Git会用`Fast forward`模式，但这种模式下，删除分支后，会丢掉分支信息。

如果要强制禁用`Fast forward`模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。

```bash 
#合并分支时，加上--no-ff参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而fast forward合并就看不出来曾经做过合并。
git merge --no-off -m "remark..." dev
```

在实际开发中，我们应该按照几个基本原则进行分支管理：

首先，`master`分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；

那在哪干活呢？干活都在`dev`分支上，也就是说，`dev`分支是不稳定的，到某个时候，比如1.0版本发布时，再把`dev`分支合并到`master`上，在`master`分支发布1.0版本；

你和你的小伙伴们每个人都在`dev`分支上干活，每个人都有自己的分支，时不时地往`dev`分支上合并就可以了。

所以，团队合作的分支看起来就像这样:

![分支策略](C:\Users\华健\Pictures\分支策略.png)

#### Bug分支

修复bug时，我们会通过创建新的bug分支进行修复，然后合并，最后删除；

当手头工作没有完成时，先把工作现场`git stash`一下，然后去修复bug，修复后，再`git stash pop`，回到工作现场。

```bash
#保存现场
git stash
#查看工作现场
git stash list
#恢复现场1:恢复的同时把stash内容也删除了
git stash pop
#恢复现场2:恢复后stash内容并不删除，需要用git stash drop来删除
git stash apply
git stash drop
```

#### 多人协作工作模式

1. 首先，可以试图用`git push origin branch-name`推送自己的修改；
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；
3. 如果合并有冲突，则解决冲突，并在本地提交；
4. 没有冲突或者解决掉冲突后，再用`git push origin branch-name`推送就能成功！

注：如果`git pull`提示“no tracking information”，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream-to origin/branch-name`  branch-name。

这就是多人协作的工作模式，一旦熟悉了，就非常简单。

- 查看远程库信息，使用`git remote -v`；

- 本地新建的分支如果不推送到远程，对其他人就是不可见的；

- 从本地推送分支，使用`git push origin branch-name`，如果推送失败，先用`git pull`抓取远程的新提交；

- 在本地创建和远程分支对应的分支，使用`git checkout -b branch-name origin/branch-name`，本地和远程分支的名称最好一致；

- 建立本地分支和远程分支的关联，使用

  ​	`git branch --set-upstream-to origin/branch-name  `branch-name ；

- 从远程抓取分支，使用`git pull`，如果有冲突，要先处理冲突。

#### GitHub Fock

- 在GitHub上，可以任意Fork开源仓库；
- 自己拥有Fork后的仓库的读写权限；
- 可以推送pull request给官方仓库来贡献代码

Bootstrap的官方仓库`twbs/bootstrap`、你在GitHub上克隆的仓库`my/bootstrap`，以及你自己克隆到本地电脑的仓库，他们的关系就像下图显示的那样：

![](C:\Users\华健\Pictures\0.png)

#### 标签管理

```bash
#新建一个标签，默认为HEAD，也可以指定一个commit id
git tag <name> <commit id>
#指定标签信息
 git tag -a <tagname> -m "ramark..."
 #用PGP签名标签
 git tag -s <tagname> -m "remark..."
 #查看所有标签
 git tag
 #查看标签信息
 git show <tagname>
 #推送一个本地标签
 git push origin <tagname>
 #推送全部未推送过的本地标签
 git push origin --tags
 #删除一个本地标签
 git tag -d <tagname>
 #删除一个远程标签
 git push origin :refs/tags/<tagname>
```

#### .gitignore文件

- 忽略某些文件时，需要编写`.gitignore`；
- `.gitignore`文件本身要放到版本库里，并且可以对`.gitignore`做版本管理

忽略文件的原则是：

1. 忽略操作系统自动生成的文件，比如缩略图等；
2. 忽略编译生成的中间文件、可执行文件等，也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，比如Java编译产生的`.class`文件；
3. 忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件。

```bash
# compiled file
*.class

# package file
*.war
*.ear

# kdiff3 ignore
*.orig

# maven ignore
target/

# eslipse ibnore
.settings/
.project
.classpath

# idea ignore
.idea/
/idea/
*.ipr
*.iml
*.iws

# temp file
*.log
*.cache
*.diff
*.patch
*.tmp

# system ignore
.DS_Store
Thumbs.db
```



#### Git配置文件

- 每个仓库的Git配置文件都放在.git/config文件中
- 而当前用户的Git配置文件放在用户主目录下的一个隐藏文件.gitconfig中


### SpringBoot

#### 入口类和@SpringBootApplication注解

```java
@SpringBootApplication
public class Ch523Application {
	public static void main(String[] args) {
		SpringApplication.run(Ch523Application.class, args);
	}
}
```

@SpringBootApplication是SpringBoot的核心注解，他是一个组合注解。

- @Configuration
- @EnableAutoConfiguration：根据类路径中的jar包依赖为当前项目进行自动配置。
- ComponentScan

关闭特定的自动配置

```java
@SpringBootApplication(exclude={DataSourceAutoConfiguration.class})
```

#### 配置文件

支持两种类型的配置文件

- application.properties

```properties
server.port=9090
server.context-path=/helloboot
```

- application.yml：是以数据为中心的语言。

```yaml
server:
	port:9090
	contextPath:/helloboot
```

《JavaEE颠覆者》附录A.3中有SpringBoot常用的配置列表。

##### 导入xml配置

SpringBoot提倡零xml配置，如果必须要使用xml配置，可以使用注解

```java
@ImportResource({"classpath:some-context.xml"})
```

@Value注解注入application.properties文件中的属性

```java
@Value("${book.author}")
private String author;
```

##### 类型安全的外部配置

通过@ConfigurationProperties加载properties文件内的配置，通过prefix属性指定properties的配置说的前缀，通过locations指定properties文件的位置：

```java
@ConfigurationProperties(prefix="author",locations={"classpath:"})
public class AuthorSettins{
    private String name;
    private String age;
    //getter、setter
}
```

#### SpringBoot默认的错误处理机制

SpringBoot针对html和客户端请求会有不同的响应。

1. 如果请求头中包含Accept: text/html参数，SpringBoot会返回默认的对应的响应状态码的错误提示页面，如404.html、500.html。也可以自定义错误页面,在/main/resources/resources/error/里新建自己的个性化错误提示页面；

2. 如果请求头中不包含Accept: text/html参数，即在客户端发送的请求，SpringBoot会返回Json格式的错误信息，如：

   ```json
   {
       "timestamp": 1524290496071,
       "status": 404,
       "error": "Not Found",
       "message": "No message available",
       "path": "/xxx"	//请求uri
   }
   ```

#### 使用 @ControllerAdvice + @ExceptionHandler 进行全局的 Controller 层异常处理

1. 自定义一个异常

   ```java
   public class UserNotExistException extends RuntimeException {

   	private static final long serialVersionUID = -6112780192479692859L;
   	private String id;
       
   	public UserNotExistException(String id) {
   		super("user not exists");
   		this.id = id;
   	}
   	public String getId() {
   		return id;
   	}
   	public void setId(String id) {
   		this.id = id;
   	}
   	public static long getSerialversionuid() {
   		return serialVersionUID;
   	}	
   }
   ```

2. 自定义Controller的异常处理类

   ```java
   @ControllerAdvice	//全局异常处理类
   public class ControllerExceptionHandler {

   	@ExceptionHandler(UserNotExistException.class)	//指定需处理的异常
   	@ResponseBody
   	@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)	//响应 状态码
   	public Map<String, Object> handleUserNotExistException(UserNotExistException ex) {
   		Map<String, Object> result = new HashMap<String, Object>();
   		result.put("id", ex.getId());
   		result.put("message", ex.getMessage());
   		return result;
   	}
   }
   ```

3. 抛出异常

   ```java
   @GetMapping("/{id:\\d+}")
   public User getInfo(@PathVariable String id) throws UserNotExistException {

       throw new UserNotExistException(id);
   }
   ```

4. 响应信息

   在客户端发送请求：http://localhost:8060/user/1

   ```json
   {
       "id": "1",
       "message": "user not exists"
   }
   ```

#### 三种拦截机制

1. 过滤器（Filter）

   SpringBoot使用Filter

   ```java
   //1.自定义（第三方）Filter
   public class TimeFilter implements Filter {

   	private Logger logger = LoggerFactory.getLogger(TimeFilter.class);
   	@Override
   	public void destroy() {
   		logger.info("TimeFilter destory");
   	}

   	@Override
   	public void doFilter(ServletRequest request, 
                            ServletResponse response, 
                            FilterChain filterChain) throws IOException, ServletException {
   		
   		Long start = new Date().getTime();
   		filterChain.doFilter(request, response);
   		Long end = new Date().getTime();
   		logger.info("服务耗时： " + (end - start));	
   		logger.info("time filter finish");
   	}

   	@Override
   	public void init(FilterConfig arg0) throws ServletException {
   		logger.info("TimeFilter init");
   	}
   }

   //2.将第三方Filter加入到过滤器链中
   @Configuration
   public class WebConfig {
   	@Bean
   	public FilterRegistrationBean filterFilter() {
   		
   		FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
   		TimeFilter timeFilter = new TimeFilter();
   		filterRegistrationBean.setFilter(timeFilter);
   		//设置该过滤器将要拦截的url
   		List<String> urls = new ArrayList<String>();
   		urls.add("/*");
   		filterRegistrationBean.setUrlPatterns(urls);
   		
   		return filterRegistrationBean;
   	}
   }
   ```

2. 拦截器（Interceptor）

   SpringBoot使用拦截器

   ```java
   //1.自定义拦截器
   @Component
   public class TimeInterceptor implements HandlerInterceptor {

   	private Logger logger = LoggerFactory.getLogger(TimeInterceptor.class);
   	
   	@Override
   	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, 
                                Object handler) throws Exception {
   		logger.info("preHandle");
   		String controllerName = ((HandlerMethod)handler).getBean().getClass().getName();
   		logger.info("controllerName : " + controllerName);
   		request.setAttribute("startTime", new Date().getTime());
   		return true;
   	}
   		
   	@Override
   	public void postHandle(HttpServletRequest request, HttpServletResponse response, 
                              Object handler, ModelAndView mav) throws Exception {
   		logger.info("postHandle");
   		Long start = (Long) request.getAttribute("startTime");
   		Long end = new Date().getTime();
   		logger.info("服务耗时 : " + (end - start));
   	}

   	@Override
   	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, 
                                   Object handler, Exception ex) throws Exception {
   		logger.info("afterCompletion");
   		Long start = (Long) request.getAttribute("startTime");
   		Long end = new Date().getTime();
   		logger.info("服务耗时 : " + (end - start));
   		logger.info("ex is : " + ex);
   	}
   }
   //2.注册拦截器，配置类需要继承WebMvcConfigurerAdapter
   @Configuration
   public class WebConfig extends WebMvcConfigurerAdapter {

   	@Autowired
   	private TimeInterceptor timeInterceptor;
   	
   	//注册拦截器
   	@Override
   	public void addInterceptors(InterceptorRegistry registry) {
   		
   		registry.addInterceptor(timeInterceptor);
   	}
   }
   ```

3. 切片（Aspect）

   ```java
   @Aspect
   @Component
   public class TimeAspect {

   	private Logger logger = LoggerFactory.getLogger(TimeAspect.class);
   	
   	//切入点
   	@Around("execution(* com.imooc.web.controller.UserController.*(..))")
   	public Object handleControllerMethod(ProceedingJoinPoint pjp) throws Throwable {
   		
   		logger.info("timeAspect start");
   		//获取方法参数
   		Object[] args = pjp.getArgs();
   		for(Object arg : args) {
   			logger.info("arg is " + arg);
   		} 
   		
   		Long start = new Date().getTime();
   		//调用被拦截的方法，Object代表被拦截的方法的返回类型
   		Object object = pjp.proceed();
   		Long end = new Date().getTime();
   		logger.info("timeAspect finish");
   		logger.info("服务耗时 ： " + (end- start));
   		return object;
   	}
   }
   ```

4. 拦截顺序

![](C:\Users\华健\Desktop\截图\拦截顺序.PNG)

代码执行结果：

```java
2018-04-22 00:49:04.571  INFO 6028 --- [nio-8060-exec-7] com.imooc.filter.TimeFilter              : time filter start
2018-04-22 00:49:04.573  INFO 6028 --- [nio-8060-exec-7] com.imooc.interceptor.TimeInterceptor    : preHandle
2018-04-22 00:49:04.574  INFO 6028 --- [nio-8060-exec-7] com.imooc.interceptor.TimeInterceptor    : controllerName : com.imooc.web.controller.UserController$$EnhancerBySpringCGLIB$$7e95a852
2018-04-22 00:49:04.577  INFO 6028 --- [nio-8060-exec-7] com.imooc.aspect.TimeAspect              : timeAspect start
2018-04-22 00:49:04.580  INFO 6028 --- [nio-8060-exec-7] com.imooc.web.controller.UserController  : 进入服务
2018-04-22 00:49:04.580  INFO 6028 --- [nio-8060-exec-7] com.imooc.aspect.TimeAspect              : timeAspect finish
2018-04-22 00:49:04.580  INFO 6028 --- [nio-8060-exec-7] com.imooc.aspect.TimeAspect              : 服务耗时 ： 3
2018-04-22 00:49:04.584  INFO 6028 --- [nio-8060-exec-7] com.imooc.interceptor.TimeInterceptor    : postHandle
2018-04-22 00:49:04.584  INFO 6028 --- [nio-8060-exec-7] com.imooc.interceptor.TimeInterceptor    : 服务耗时 : 10
2018-04-22 00:49:04.584  INFO 6028 --- [nio-8060-exec-7] com.imooc.interceptor.TimeInterceptor    : afterCompletion
2018-04-22 00:49:04.584  INFO 6028 --- [nio-8060-exec-7] com.imooc.interceptor.TimeInterceptor    : 服务耗时 : 10
2018-04-22 00:49:04.584  INFO 6028 --- [nio-8060-exec-7] com.imooc.interceptor.TimeInterceptor    : ex is : null
2018-04-22 00:49:04.584  INFO 6028 --- [nio-8060-exec-7] com.imooc.filter.TimeFilter              : time filter finish
2018-04-22 00:49:04.584  INFO 6028 --- [nio-8060-exec-7] com.imooc.filter.TimeFilter              : 服务耗时 ： 13
```

5. SpringMvc的执行流程

![](C:\Users\华健\Desktop\截图\SpringMvc执行流程.png)

### Linux

#### centos6.8 配置网络

桥接模式

1.配置ip地址：vi /etc/sysconfig/network-scripts/ifcfg-eth0

> DEVICE=eth0 （网卡名称）
>
> TYPE=Ethernet
>
> ONBOOT=yes （是否开机自启动）
>
> NM_CONTROLLED=yes
>
> BOOTPROTO=static  （获取ip方式，有静态，动态dhpd，和不指定none）
>
> IPADDR=192.168.1.105 （ip地址，先与本地cmd -》ipconfig 查看 本地ip，然后改变其于同一网段或尾数往上加，并与本地进行ping测试，不能访问既可用）
>
> NETMASK=255.255.255.0  （查看本地cmd -》ipconfig）
>
> GATEWAY=192.168.0.1  （查看本地cmd -》ipconfig）
>
> USERCTL=no  （是否非用户可以访问）
>
> DNS1=114.114.114.114 (DNS在本文件中设置也可，每次重启网络，/etc/resolv.conf中dns设置将被冲洗)
>
> DNS2=8.8.8.8

2.配置DNS     vi /etc/resolv.conf    

> > nameserver xxxxxx  ----本地ip（没设置这个，也能访问到）
>
> > nameserver 114.114.114.114 ----中国三大巨头，全网通
>
> > nameserver 8.8.8.8---谷歌全球通

**3.配置网关：**vi /etc/sysconfig/network

> > NETWORKING=yes
>
> > HOSTNAME=localhost.localdomai
>
> > GATEWAY=192.168.0.1（与1的设置对应）

4.重启网络   service network restart

   出现 l绿色 ok 四个既可

5.测试  ping www.baidu.com 测试下  是否能联网

#### Linux防火墙

​	防火墙策略可以基于流量的源目地址、端口号、协议、应用等信息来定制，然后防火墙使用预先定制的策略规则监控出入的流量，若流量与某一条策略规则相匹配，则执行相应的处理，反之则丢弃。 

​	iptables 与 firewalld 都不是真正的防火墙，它们都只是用来定义防火墙策略的防火墙管理工具而已，或者说，它们只是一种服务。 iptables服务会把配置好的防火墙策略交由内核层面的 netfilter 网络过滤器来处理，而 firewalld 服务则是把配置好的防火墙策略交由内核层面的 nftables 包过滤框架来处理。 

​	一般而言，防火墙策略规则的设置有两种：一种是“通”（即放行），一种是“堵”（即阻止）。当防火墙的默认策略为拒绝时（堵），就要设置允许规则（通），否则谁都进不来；如果防火墙的默认策略为允许时，就要设置拒绝规则，否则谁都能进来，防火墙也就失去了防范的作用。 

​	iptables 服务把用于处理或过滤流量的策略条目称之为规则，多条规则可以组成一个规则链，而规则链则依据数据包处理位置的不同进行分类，具体如下：

➢ 在进行路由选择前处理数据包（PREROUTING）；
➢ 处理流入的数据包（INPUT）；
➢ 处理流出的数据包（OUTPUT）；
➢ 处理转发的数据包（FORWARD）；
➢ 在进行路由选择后处理数据包（POSTROUTING）。 

​	防火墙策略规则的匹配顺序是从上至下的，因此要把较为严格、优先级较高的策略规则
放到前面，以免发生错误。 防火墙策略规则的动作：

- ACCEPT（允许流量通过） 
- LOG（记录日志信息） 
- REJECT（拒绝流量通过，并且响应，Destination Port Unreachable ） 
- DROP（拒绝流量通过，直接丢弃且不响应，流量发送方会受到响应超时的提示） 

##### iptables中常用的参数及作用

|    参数     |                      作用                       |
| :---------: | :---------------------------------------------: |
|     -P      |                  设置默认策略                   |
|     -F      |                   清空规则链                    |
|     -L      |                   查看规则链                    |
|     -A      |            在规则链的末尾加入新规则             |
|   -I num    |            在规则链的头部加入新规则             |
|   -D num    |                 删除某一条规则                  |
|     -s      | 匹配来源地址 IP/MASK，加叹号“!”表示除这个 IP 外 |
|     -d      |                  匹配目标地址                   |
| -i 网卡名称 |            匹配从这块网卡流入的数据             |
| -o 网卡名称 |            匹配从这块网卡流出的数据             |
|     -p      |          匹配协议，如 TCP、 UDP、 ICMP          |
| --dport num |                 匹配目标端口号                  |
| --sport num |                 匹配来源端口号                  |

注1：规则配置文件在**/etc/sysconfig/iptables**中，可以直接修改文件或使用iptables命令修改规则。

注2：默认的两条规则要放在规则链的最后

```bash
#这两条的意思是在INPUT表和FORWARD表中拒绝所有其他不符合上述任何一条规则的数据包。并且发送一条host prohibited的消息给被拒绝的主机。一定要放在（/etc/sysconfig/iptables文件）的最后,否则会覆盖掉后面的规则。
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
```

#### TODO:SeLinux

#### centos6使用阿里云yum源配置

```bash
#1.备份
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
#2.下载新的CentOS-Base.repo 到/etc/yum.repos.d/
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
#3.之后运行yum makecache生成缓存
yum makecache
```

#### Linux安装JDK

```bash
#1.查看并清理系统默认自带jdk
rpm -qa | grep jdk （管道命令符：把前一个命令原本要输出到屏幕的数据当作是后一个命令的标准输入）
sudo yum remove XXX（XXX为上一个命令查到的结果）
#2.赋予文件权限
sudo chmod 777 jdk-8u162-linux-x64.rpm
#3.下载jdk,在哪个路径执行wget就下载在哪个位置
wget http://download.oracle.com/otn-pub/java/jdk/8u162-	   
		b12/0da788060d494f5095bf8624735fa2f1/jdk-8u162-linux-x64.rpm?
		AuthParam=1523427758_49f6707315e6a98811caba4554f948a9			
#4.安装，默认安装路径/usr/java
sudo rpm -ivh jdk-8u162-linux-x64.rpm
			AuthParam=1523427758_49f6707315e6a98811caba4554f948a9
#5.jdk配置环境变量
sudo vim /etc/profile
	export JAVA_HOME=/usr/java/jdk1.8.0_162
	export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar
    		         :$JAVA_HOME/lib/tools.jar
     export PATH=$JAVA_HOME/bin:$PATH
#6.使配置生效,source命令通常用于重新执行刚修改的初始化文件，使之立即生效，不必注销并重新登录。
source /etc/profile  
#7.验证
java -version
```

#### Linux安装Tomcat

```bash
#1.下载,一般用户程序放在/usr/local目录中
wget 官网地址
#2.解压缩（-z:用 Gzip 压缩或解压;-x:解压;-v:显示进度;-f:目标文件名）
tar -zxvf apache-tomcat-8.5.30.tar.gz
#3.配置环境变量
export CATALINA_HOME=/usr/local/apache-tomcat-8.5.30
#4.使配置生效
source /etc/profile  
#5.配置UTF-8字符集：进入到${CATALINA_HOME}/conf/server.xml文件中，找到配置8080默认端口的位置，在xml节点末尾增加URIEncoding="UTF-8"
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" URIEncoding="UTF-8" />
#验证
#进入到tomcat的bin目录，执行./startup.sh(输入完整路径执行批处理文件),看到如下信息表示启动成功
[root@centos6 bin]# ./startup.sh
Using CATALINA_BASE:   /usr/local/apache-tomcat-8.5.30
Using CATALINA_HOME:   /usr/local/apache-tomcat-8.5.30
Using CATALINA_TMPDIR: /usr/local/apache-tomcat-8.5.30/temp
Using JRE_HOME:        /usr/java/jdk1.8.0_162
Using CLASSPATH:       /usr/local/apache-tomcat-8.5.30/bin/bootstrap.jar:/usr/local/apache-tomcat-8.5.30/bin/tomcat-juli.jar
Tomcat started.
#访问,如无法访问可能是防火墙的问题,关闭防火墙：service iptables stop
http://虚拟机ip:8080
```

#### Linux安装Maven

```bash
#1.下载
wget 官网地址
#2.解压缩
tar -zxvf apache-maven-3.5.3
#3.配置环境变量
export MAVEN_HOME=/usr/local/apache-maven-3.5.3
export PATH=$MAVEN_HOME/bin:$JAVA_HOME/bin:$PATH
#4.使配置生效
source /etc/profile
#5.验证
mvn -version
#6.修改中央仓库为阿里云镜像仓库，/conf/settings.xml中加上
<mirror>  
	<id>alimaven</id>  
	<name>aliyun maven</name>  
	<url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
	<mirrorOf>central</mirrorOf>          
</mirror> 
```

#### Linux安装vsftpd

vsftpd是“very secure FTP daemon”的缩写，是一个完全免费的、开放源代码的ftp服务器软件。

vsftpd是一款在Linux发行版中最受推崇的FTP服务器程序，小而轻快，安全易用，支持虚拟用户、支持贷款限制等功能。

##### vsftpd 三种认证模式登录到 FTP 服务器上

➢ 匿名开放模式：是一种最不安全的认证模式，任何人都可以无需密码验证而直接登录到 FTP 服务器。

➢ 本地用户模式：是通过 Linux 系统本地的账户密码信息进行认证的模式，相较于匿名开放模式更安全，而且配置起来也很简单。但是如果被黑客破解了账户的信息，就可以畅通无阻地登录 FTP 服务器，从而完全控制整台服务器。

➢ 虚拟用户模式：是这三种模式中最安全的一种认证模式，它需要为 FTP 服务单独建立用户数据库文件，虚拟出用来进行口令验证的账户信息，而这些账户信息在服务器系统中实际上是不存在的，仅供 FTP 服务程序进行认证使用。这样，即使黑客破解了账户信息也无法登录服务器，从而有效降低了破坏范围和影响。 

ftp常用命令

```bash
service vsftpf start	#启动
service vsftpf stop		#关闭
service vsftpf restart	#重启
```

安装

```bash
#1.检查是否已安装vsftpd
rpm -qa | grep vsftpd
#2.安装vsftpd,默认配置文件在/etc/vsftpd/vsftpd.conf
yum install vsftpd
```

创建虚拟用户

```bash
#1.选择在根或者用户目录下擦混关键ftp文件夹
mkdir /ftpfile
#2.添加匿名用户 (-d:指定用户的家目录；-s:指定该用户的默认 Shell 解释器;/sbin/nologin表示不能登录到系统中)
useradd ftpuser -d /ftpfile -s /sbin/nologin
#3.修改ftpfile权限
chown -R ftpuser.ftpuser /ftpfile
#4.重设ftpuser密码（123456）
passwd ftpuser
```

配置

```bash
#1.新建配置文件chroot_list
cd /etc/vsftpd
vim chroot_list
#2.把刚才新增的虚拟用户添加到此配置文件中，后续要引用
#3.修改配置文件/etc/selinux/config
vim /etc/selinux/config
set seforce 0  	#临时关闭seLinux
#注：如果验证的时候碰到550拒绝访问请执行：
sudo setsebool -P ftp_home_dir 1
#4.重启Linux
reboot
#5.配置/etc/vsftpd/vsftpd.conf
本项目要用到的配置项：
1）local_root=/ftpfile(当本地用户登入时，将被更换到定义的目录下，默认值为各用户的家目录) 
2）anon_root=/ftpfile(使用匿名登入时，所登入的目录) 
3）use_localtime=YES(默认是GMT时间，改成使用本机系统时间)
4）anonymous_enable=NO(不允许匿名用户登录)
5）local_enable=YES(允许本地用户登录)
6）write_enable=YES(本地用户可以在自己家目录中进行读写操作)
7）local_umask=022(本地用户新增档案时的umask值)
8）dirmessage_enable=YES(如果启动这个选项，那么使用者第一次进入一个目录时，会检查该目录下是否有.message这个档案，如果有，则会出现此档案的内容，通常这个档案会放置欢迎话语，或是对该目录的说明。默认值为开启)
9）xferlog_enable=YES(是否启用上传/下载日志记录。如果启用，则上传与下载的信息将被完整纪录在xferlog_file 所定义的档案中。预设为开启。)
10）connect_from_port_20=YES(指定FTP使用20端口进行数据传输，默认值为YES)
11）xferlog_std_format=YES(如果启用，则日志文件将会写成xferlog的标准格式)
12）ftpd_banner=Welcome to mmall FTP Server(这里用来定义欢迎话语的字符串)
13）chroot_local_user=NO(用于指定用户列表文件中的用户是否允许切换到上级目录)
14）chroot_list_enable=YES(设置是否启用chroot_list_file配置项指定的用户列表文件)
15）chroot_list_file=/etc/vsftpd/chroot_list(用于指定用户列表文件)
16）listen=YES(设置vsftpd服务器是否以standalone模式运行，以standalone模式运行是一种较好的方式，此时listen必须设置为YES，此为默认值。建议不要更改，有很多与服务器运行相关的配置命令，需要在此模式下才有效，若设置为NO，则vsftpd不是以独立的服务运行，要受到xinetd服务的管控，功能上会受到限制)
17）pam_service_name=vsftpd(虚拟用户使用PAM认证方式，这里是设置PAM使用的名称，默认即可，与/etc/pam.d/vsftpd对应) 
userlist_enable=YES(是否启用vsftpd.user_list文件，黑名单,白名单都可以
18)pasv_min_port=61001(被动模式使用端口范围最小值)
19)pasv_max_port=62000(被动模式使用端口范围最大值)
20)pasv_enable=YES(pasv_enable=YES/NO（YES）
若设置为YES，则使用PASV工作模式；若设置为NO，则使用PORT模式。默认值为YES，即使用PASV工作模式。
#6.防火墙配置
vim /etc/sysconfig/iptables
#vsftpd
-A INPUT -p TCP --dport 61001:62000 -j ACCEPT
-A OUTPUT -p TCP --sport 61001:62000 -j ACCEPT
-A INPUT -p TCP --dport 20 -j ACCEPT
-A OUTPUT -p TCP --sport 20 -j ACCEPT
-A INPUT -p TCP --dport 21 -j ACCEPT
-A OUTPUT -p TCP --sport 21 -j ACCEPT
#7.重启ftp服务
service vsftpd restart
#8.验证：浏览器输入
ftp://虚拟机ip
```

#### windows安装ftp服务

1. 安装ftpserver绿色版。
2. 在浏览器中输入ftp://本机ip进行验证。
3. 安装ftp客户端软件，如:FileZilla，WinScp等。

#### FTP协议

​	FTP 是一种在互联网中进行文件传输的协议，基于客户端/服务器模式，默认使用 20、 21
号端口，其中端口 20（数据端口）用于进行数据传输，端口 21（命令端口）用于接受客户端
发出的相关 FTP 命令与参数。 

##### FTP协议有两种工作方式：

​	PORT方式（主动式）和PASV方式（被动式）。

1. PORT（主动）方式的连接过程是：客户端向服务器的FTP端口（默认是21）发送连接请求，服务器接受连接，建立一条命令链路。 当需要传送数据时，客户端在命令链路上用 PORT命令告诉服务器：“我打开了端口，你过来连接我”。于是服务器从20端口向客户端的端口发送连接请求，建立一条数据链路来传送数据。


2. PASV（被动）方式的连接过程是：客户端向服务器的FTP端口（默认是21）发送连接请求，服务器接受连接，建立一条命令链路。 当需要传送数据时，服务器在命令链路上用 PASV命令告诉客户端：“我打开了端口，你过来连接我”。于是客户端向服务器的端口发送连接请求，建立一条数据链路来传送数据。 

  	从上面可以看出，两种方式的命令链路连接方法是一样的，而数据链路的建立方法就完全不同。而FTP的复杂性就在于此。


#### Linux源码安装软件步骤

1. 解压缩

		tar -zxvf  xxx.tar.gz

2. 配置

      cd xxx

      ./configure

3. 编译 

​        make

4. 安装

​        make install

5. 卸载

   ​    make uninstall

#### Linux安装Nginx

Nginx是一款用于部署动态网站的轻量级Web服务器，因其稳定性、功能丰富、占用内存少且并发能力强而备受用户的信赖。 

##### Nginx功能：

1. 可直接支持Rails和PHP的程序性
2. 可作为HTTP反向代理服务器
3. 作为负载均衡服务器
4. 作为邮件代理服务器
5. 帮助实现前端动静分离

##### Nginx特点

1. 高稳定
2. 高性能
3. 资源占用少
4. 功能丰富
5. 模块化结构
6. 支持热部署

##### 安装

```bash
#1.安装gcc，输入gcc -v查询版本信息，看是否系统自带
yum install gcc
#2.安装pcre
yum install pcre-devel
#3.安装zlib
yum install zlib zlib-devel
#4.安装openssl(如需支持ssl,才安装openssl)
yum install openssl openssl-devel
#5.下载源码包，选择稳定版本，解压缩安装
wget http://nginx.org/download/nginx-1.10.3.tar.gz
tar -zxvf nginx-1.10.3.tar.gz
#6.Nginx安装
#进入nginx目录之后执行
./configure	#配置
#也可以指定安装目录，增加参数--prefix=/usr/nginx
#如果不指定路径，可以通过whereis nginx进行查询
#默认安装在/usr/local/nginx
#继续执行
make	#编译
make install	#安装
#6.验证,访问
http://ip:80 #（默认80端口）
#7.防火墙配置
sudo vim /etc/sysconfig/iptables
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
service iptables restart
```

##### Nginx反向代理配置及测试验证

```bash
#1.编辑配置文件
vim ${nginx}/conf/nginx.conf
#增加一句，表示引入vhost文件夹下的所有*.conf文件。目的是避免nginx.conf文件变得庞大臃肿
include vhost/*.conf;
#2.创建vHost文件夹
mkdir vHost
#3.新建代理配置文件
vim www.imooc.com.conf
#4.Linux配置host
vim /etc/hosts
server {  
	listen 80; 
	autoindex on; 
	server_name www.imooc.com; 
	access_log /usr/local/nginx/logs/access.log combined; 
	index index.html index.htm index.jsp index.php; 
	#error_page 404 /404.html; 
	if ( $query_string ~* ".*[\;'\<\>].*" ){ 
		return 404; 
	} 
	location / { 
		proxy_pass http://127.0.0.1:8080;	#转发到ip地址
		#root /ftpfile/; 				   #转发到目录
		#root c:\ftpfile\img;			   #windows
		add_header Access-Control-Allow-Origin *; 
	} 
}
#5.添加对应的域名及ip,避免实验时转发到真实的网址，如：
192.168.0.8 www.imooc.com
192.168.0.8 image.imooc.com
192.268.0.8 s.imooc.com
#6.访问www.imooc.com测试
```

##### Nignx命令

```bash
#测试配置文件
安装路径下的/nginx/sbin/nginx -t
#启动命令
安装路径下的/nginx/sbin/nginx
#停止命令
安装路径下的/nginx/sbin/nginx -s stop
安装路径下的/nginx/sbin/nginx -s quit
#重启命令
安装路径下的/nginx/sbin/nginx -s reload
#查看进程命令
ps -ef | grep nginx
#平滑重启
kill -HUB [Nginx主进程号（即查看进程命令查到的PID）]
#注：配置环境变量，不要输全路径
```

#### Windows安装Nginx

1. 下载http://nginx.org/download/nginx-1.10.3.zip
2. 解压缩
3. 运行nainx.exe，通过双击图标或者cmd命令行运行

4.windows配置hosts

​	路径：‪C:\Windows\System32\drivers\etc\hosts

5.配置文件同Linux

#### Linux安装MySql

```bash
#1.使用阿里的yum源安装
yum install mysql-server
#2.检查是否已经安装
rpm -qa | grep mysql-server
#默认配置文件在/etc/my.cnf
#3.字符集配置
vim /etc/my.cnf
#在[mysqld]节点下添加：
default-character-set=utf8
character-set-server=utf8
#4.自启动配置
chkconfig mysqld on
chkconfig --list mysqld #查看（如果2-5位启用on状态即OK）
#5.防火墙配置并重启
-A INPUT -p tcp -m tcp --dport 3306 -j ACCEPT
#6.启动mysql服务
service mysqld restart
#7.登录mysql服务
mysql -u root
```

windows中mysql字符集配置

1. 找到my.ini文件，查找default-character-set节点，修改成utf8
2. 查找character-set-server节点，修改成utf8

MySQL配置

```bash
#注：语句结尾一定要加分号
#查看目前musql的用户
select user.host,password from mysql.user;
#修改root密码
set password for root@127.0.0.1=password('yourpassword');
#退出mysql
exit
#密码登录
mysql -u root -p
#查看是否有匿名用户
select user,host from mysql.user;
#删除匿名用户
delete from mysql.user where user = '';
#插入mysql新用户
insert into mysql.user(Host,User,Password) values("localhost","yourname",password("yourpassword"));
#使操作生效
flush privileges;
#创建新的database。collate utf8_general_ci：数据库的校验规则，ci指大小写不敏感。
create database `mmall` default character set utf8 collate utf8_general_ci;
#本地用户赋予所有权限
grant all privileges on mmall.* to 'huajian'@'localhost' identified by '123456';
#给账号开通外网所有权限
grant all privileges on mmall.* to 'huajian'@'%' identified by '123456';
```

#### Linux安装Git

```bash
#1.安装依赖
yum install zlib-devel openssl-devel cpio expat-devel gettext-devel curl-devel perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker
#1.下载git
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.17.0.tar.gz
#3.解压缩
tar -zxvf git-2.17.0
#4.配置,用来检测你的安装平台的目标特征的。
cd git-2.17.0
./configure
#5.编译,从Makefile中读取指令，然后编译。
make
#6.安装,从Makefile中读取指令，安装到指定的位置。
make install
#7.验证
git --version
```

Git配置

```bash
#1.配置用户名
git config --global user.name "huajian"
#2.配置邮箱
git config --global user.email "hero514346219@163.com"
#3.让Git不要管Windows、Unix换行符转换的事
git config --global core.autocrlf false
#4.编码配置
git config --global gui.encoding utf-8	#避免git gui中的中文乱码
git config --global core.quetepath off  #避免git status显示的中文文件名乱码
#windows上还需配置
git config --global core.ignorecase false
```

Git ssh key pair配置

```bash
#生成密钥对，文件存放在用户家目录的.ssh文件夹中
ssh-keygen -t rsa -C "hero514346219@163.com"
#若报错，先执行`ssh-agent`,再执行下面说的命令，用ssh-add -l查看新加的rsa
ssh-add ~/.ssh/id_rsa
#查看公钥，添加到github、gitlab、码云
cat ~/.ssh/id_rsa.pub
```

#### Linux自动部署

编写自动部署脚本：autodeploy.sh

```bash
echo "========apache-tomcat-8.5.30===进入git项目happymmall目录============="
cd /develop/git-repository/mmall_learning

echo "==========git切换分之到mmall-v1.0==============="
git checkout v1.0

echo "==================git fetch======================"
git fetch

echo "==================git pull======================"
git pull

echo "===========编译并跳过单元测试===================="
mvn clean package -Dmaven.test.skip=true

echo "============删除旧的ROOT.war==================="
rm /usr/local/apache-tomcat-8.5.30/webapps/ROOT.war

echo "======拷贝编译出来的war包到tomcat下-ROOT.war======="
cp /develop/git-repository/mmall_learning/target/mmall.war  /usr/local/apache-tomcat-8.5.30/webapps/ROOT.war

echo "============删除tomcat下旧的ROOT文件夹============="
rm -rf /usr/local/apache-tomcat-8.5.30/webapps/ROOT

echo "====================关闭tomcat====================="
/usr/local/apache-tomcat-8.5.30/bin/shutdown.sh

echo "================sleep 10s========================="
for i in {1..10}
do
        echo $i"s"
        sleep 1s
done

echo "====================启动tomcat====================="
/usr/local/apache-tomcat-8.5.30/bin/startup.sh
```

#### IDEA中终端Terminal使用Linux Shell

1. 打开设置（快捷键：` Ctrl + Alt + S `），进入 `Plugins`,搜索栏搜索 `Terminal`，查看 `Terminal`插件是否打勾选中，如果没有，请打勾。

2. 进入设置（快捷键：Ctrl + Alt + S），进入 `Tools`字段，再进入 `Terminal`字段，在 `Shell path`那一栏中，输入你主机 `Git Bash`的安装位置

   ```bash
   "路径\Git\bin\sh.exe" -login -i	#不要忘记冒号
   ```

3.重启

#### 既使用maven编译，又使用lib下的jar包编译

​	一般情况下jar包都可以使用pom.xml来配置管理，但也有一些时候，我们项目中使用了一个内部jar文件，但是这个文件我们又没有开放到maven库中。 我们会将文件当到我们项目WEB-INF/lib中。 如果我们不对pom.xml进行特殊配置的话，maven打包是不会自动去引用和编译lib中的jar文件的，所以需要我们修改下pom.xml文件。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <encoding>UTF-8</encoding>
        <compilerArguments>
            <extdirs>${project.basedir}/src/main/webapp/WEB-INF/lib</extdirs>
        </compilerArguments>
    </configuration>
</plugin>
```

#### IDEA使用mybatis generator插件遇到的坑

1. 新建配置文件选择New Generator File，否则右键文件看不到Run Mybatis Generator，如图：

![](C:\Users\华健\Pictures\new mybatis config.png)

![](C:\Users\华健\Pictures\右键run mybatis.png)

2. 找不到mysql驱动类jar包位置

直接在generatorConfig.xml的classPathEntry中填写全路径，如在datasource.xml中配置全路径，运行时会报错：找不到jar包位置。

```
<!--指定特定数据库的jdbc驱动jar包的位置-->
<classPathEntry location="D:\mavenRepo\mysql\mysql-connector-java\5.1.6\mysql-connector-java-5.1.6.jar"/>
```

3. 引入数据库配置文件会报错

![](C:\Users\华健\Pictures\datasource err.png)

```
<!--导入属性配置,会报错，注释掉，直接在generatorConfig.xml中进行相关配置-->
<properties resource="datasource.properties" />
```

#### 典型的SSM项目配置文件(Spring官网demo)

##### web.xml

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >
<web-app>
    <display-name>Archetype Created Web Application</display-name>
    <!--字符集编码过滤器-->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <!--Web容器启动和关闭的监听器-->
    <listener>
        <listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
    </listener>
    <!--Web容器与Spring容器整合的监听器-->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath:applicationContext.xml
        </param-value>
    </context-param>
    <!--SpringMvc配置-->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:dispatcher-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
</web-app>
```

##### applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
     http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
     http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
     http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
     http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--开启扫描-->
    <context:component-scan base-package="com.mmall" annotation-config="true"/>
    <!--开启aop代理-->
    <!--<context:annotation-config/>-->
    <aop:aspectj-autoproxy/>
    <!--导入数据库相关配置-->
    <import resource="applicationContext-datasource.xml"/>
</beans>
```

applicationContext-dataSource.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
     http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
     http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
     http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
     http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--开启包扫描-->
    <context:component-scan base-package="com.mmall" annotation-config="true"/>

    <!--加载数据库常量配置文件-->
    <bean id="propertyConfigurer"
          class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="order" value="2"/>
        <property name="ignoreUnresolvablePlaceholders" value="true"/>
        <property name="locations">
            <list>
                <value>classpath:datasource.properties</value>
            </list>
        </property>
        <property name="fileEncoding" value="utf-8"/>
    </bean>


    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${db.driverClassName}"/>
        <property name="url" value="${db.url}"/>
        <property name="username" value="${db.username}"/>
        <property name="password" value="${db.password}"/>
        <!-- 连接池启动时的初始值 -->
        <property name="initialSize" value="${db.initialSize}"/>
        <!-- 连接池的最大值 -->
        <property name="maxActive" value="${db.maxActive}"/>
        <!-- 最大空闲值.当经过一个高峰时间后，连接池可以慢慢将已经用不到的连接慢慢释放一部分，一直减少到maxIdle为止 -->
        <property name="maxIdle" value="${db.maxIdle}"/>
        <!-- 最小空闲值.当空闲的连接数少于阀值时，连接池就会预申请去一些连接，以免洪峰来时来不及申请 -->
        <property name="minIdle" value="${db.minIdle}"/>
        <!-- 最大建立连接等待时间。如果超过此时间将接到异常。设为－1表示无限制 -->
        <property name="maxWait" value="${db.maxWait}"/>
        <!--#给出一条简单的sql语句进行验证 -->
         <!--<property name="validationQuery" value="select getdate()" />-->
        <property name="defaultAutoCommit" value="${db.defaultAutoCommit}"/>
        <!-- 回收被遗弃的（一般是忘了释放的）数据库连接到连接池中 -->
         <!--<property name="removeAbandoned" value="true" />-->
        <!-- 数据库连接过多长时间不用将被视为被遗弃而收回连接池中 -->
         <!--<property name="removeAbandonedTimeout" value="120" />-->
        <!-- #连接的超时时间，默认为半小时。 -->
        <property name="minEvictableIdleTimeMillis" value="${db.minEvictableIdleTimeMillis}"/>

        <!--# 失效检查线程运行时间间隔，要小于MySQL默认-->
        <property name="timeBetweenEvictionRunsMillis" value="40000"/>
        <!--# 检查连接是否有效-->
        <property name="testWhileIdle" value="true"/>
        <!--# 检查连接有效性的SQL语句-->
        <property name="validationQuery" value="SELECT 1 FROM dual"/>
    </bean>

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="mapperLocations" value="classpath*:mappers/*Mapper.xml"></property>

        <!-- 分页插件 -->
        <property name="plugins">
            <array>
                <bean class="com.github.pagehelper.PageHelper">
                    <property name="properties">
                        <value>
                            dialect=mysql
                        </value>
                    </property>
                </bean>
            </array>
        </property>

    </bean>

    <!--开启mybatis的dao层扫描-->
    <bean name="mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.mmall.dao"/>
    </bean>

    <!-- 使用@Transactional进行声明式事务管理需要声明下面这行 -->
    <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true" />
    <!-- 事务管理 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
        <property name="rollbackOnCommitFailure" value="true"/>
    </bean>
</beans>
```

##### dispatcherServlet.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/mvc
	http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--开启包扫描-->
    <context:component-scan base-package="com.mmall" annotation-config="true"/>

    <mvc:annotation-driven>
        <mvc:message-converters>
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <property name="supportedMediaTypes">
                    <list>
                        <value>text/plain;charset=UTF-8</value>
                        <value>text/html;charset=UTF-8</value>
                    </list>
                </property>
            </bean>
            <bean class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter">
                <property name="supportedMediaTypes">
                    <list>
                        <value>application/json;charset=UTF-8</value>
                    </list>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>

    <!-- 文件上传 -->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="maxUploadSize" value="10485760"/> <!-- 10m -->
        <property name="maxInMemorySize" value="4096" />
        <property name="defaultEncoding" value="UTF-8"></property>
    </bean>
</beans>
```

##### datasource.properties

```properties
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
    <display-name>Archetype Created Web Application</display-name>

    <!--字符集编码过滤器-->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--Web容器启动和关闭的监听器-->
    <listener>
        <listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
    </listener>

    <!--Web容器与Spring容器整合的监听器-->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath:applicationContext.xml
        </param-value>
    </context-param>

    <!--SpringMvc配置-->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:dispatcher-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>

</web-app>
```

##### logback.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoding>UTF-8</encoding>
        <encoder>
            <pattern>[%d{HH:mm:ss.SSS}][%p][%c{40}][%t] %m%n</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>DEBUG</level>
        </filter>
    </appender>

    <appender name="mmall" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--<File>d:/mmalllog/mmall.log</File>-->
        <File>/developer/apache-tomcat-7.0.73/logs/mmall.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>/developer/apache-tomcat-7.0.73/logs/mmall.log.%d{yyyy-MM-dd}.gz</fileNamePattern>
            <append>true</append>
            <maxHistory>10</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>[%d{HH:mm:ss.SSS}][%p][%c{40}][%t] %m%n</pattern>
        </encoder>
    </appender>


    <appender name="error" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--<File>d:/mmalllog/error.log</File>-->
        <File>/developer/apache-tomcat-7.0.73/logs/error.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>/devsoft/apache-tomcat-7.0.73/logs/error.log.%d{yyyy-MM-dd}.gz</fileNamePattern>
            <!--<fileNamePattern>d:/mmalllog/error.log.%d{yyyy-MM-dd}.gz</fileNamePattern>-->
            <append>true</append>
            <maxHistory>10</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>[%d{HH:mm:ss.SSS}][%p][%c{40}][%t] %m%n</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <logger name="com.mmall" additivity="false" level="INFO" >
        <appender-ref ref="mmall" />
        <appender-ref ref="console"/>
    </logger>

    <!-- geelynote mybatis log 日志 -->

    <logger name="com.mmall.dao" level="DEBUG"/>

    <!--<logger name="com.ibatis.sqlmap.engine.impl.SqlMapClientDelegate" level="DEBUG" >-->
        <!--<appender-ref ref="console"/>-->
    <!--</logger>-->

    <!--<logger name="java.sql.Connection" level="DEBUG">-->
        <!--<appender-ref ref="console"/>-->
    <!--</logger>-->
    <!--<logger name="java.sql.Statement" level="DEBUG">-->
        <!--<appender-ref ref="console"/>-->
    <!--</logger>-->

    <!--<logger name="java.sql.PreparedStatement" level="DEBUG">-->
        <!--<appender-ref ref="console"/>-->
    <!--</logger>-->

    <root level="DEBUG">
        <appender-ref ref="console"/>
        <appender-ref ref="error"/>
    </root>
</configuration>
```

### 用户模块开发

#### 横向越权与纵向越权安全漏洞

- 横向越权：攻击者尝试访问与其他拥有相同权限的用户的资源
- 纵向越权：低级别攻击者尝试访问高级别用户的资源

#### 高复用服务响应对象的抽象封装

响应对象：

```java
import net.sf.jsqlparser.schema.Server;
import org.codehaus.jackson.annotate.JsonIgnore;
import org.codehaus.jackson.map.annotate.JsonSerialize;
import java.io.Serializable;

@JsonSerialize(include = JsonSerialize.Inclusion.NON_NULL)
//保证序列化json的时候，如果是null的对象，key也会消失
public class ServerResponse<T> implements Serializable {

    private int status;		//响应码
    private String msg;		//响应信息
    private T data;			//响应数据

    private ServerResponse(int status) {
        this.status = status;
    }
    private ServerResponse(int status, String msg) {
        this.status = status;
        this.msg = msg;
    }
    private  ServerResponse(int status, T data) {
        this.status = status;
        this.data = data;
    }
    private ServerResponse(int status, String msg, T data) {
        this.status = status;
        this.data = data;
        this.data = data;
    }

    @JsonIgnore
    //使之不在json序列化结果当中
    public boolean isSuccess() {
        return this.status == ResponseCode.SUCCESS.getCode();
    }
    public int getStatus() {
        return status;
    }
    public String getMsg() {
        return msg;
    }
    public T getData() {
        return data;
    }

    public static <T> ServerResponse<T> createBySuccess() {
        return new ServerResponse<T>(ResponseCode.SUCCESS.getCode());
    }
    public static <T> ServerResponse<T> createBySuccessMessage(String message) {
        return new ServerResponse<T>(ResponseCode.SUCCESS.getCode(), message);
    }
    public static <T> ServerResponse<T> createBySuccess(T data) {
        return new ServerResponse<T>(ResponseCode.SUCCESS.getCode(), data);
    }
    public static <T> ServerResponse<T> createBySuccess(String msg, T data) {
        return new ServerResponse<T>(ResponseCode.SUCCESS.getCode(), msg, data);
    }

    public static <T> ServerResponse<T> createByError() {
        return new ServerResponse<T>(ResponseCode.ERROR.getCode());
    }
    public static <T> ServerResponse<T> createByErrorMessage(String message) {
        return new ServerResponse<T>(ResponseCode.ERROR.getCode(), message);
    }

    public static <T> ServerResponse<T> createByErrorCodeMessage(int errorCode, String message) {
        return new ServerResponse<T>(errorCode, message);
    }
}
```

响应码枚举：

```java
public enum ResponseCode {

    SUCCESS(0, "SUCCESS"),
    ERROR(1, "ERROR"),
    NEED_LOGIN(10, "NEED_LOGIN"),
    ILLEGAL_ARGUMENT(2, "ILLEGAL_ARGUMENT");

    private final int code;
    private final String desc;

    private ResponseCode(int code, String desc){
        this.code = code;
        this.desc = desc;
    }

    public int getCode() {
        return code;
    }

    public String getDesc() {
        return desc;
    }
}
```

#### Guava cache的使用

用户忘记密码的时候，希望重置密码，需要先验证关于密码问题的答案。步骤如下：

1. 获取关于密码的问题；
2. 获得问题，前台填写答案，发送到后台进行验证；
3. 如答案验证成功，将一个token(UUID就行)放入到缓存中（Guava Cache实现），并将token发送到前台；
4. 前台填写新的密码，携带token返回到后台；
5. 后台从缓存中取出之前存入的token，与前台提交的token进行对比；
6. 如对比成功，则更新密码。

其中token使用Guava工具类进行缓存的操作。

```java
import com.google.common.cache.CacheBuilder;
import com.google.common.cache.CacheLoader;
import com.google.common.cache.LoadingCache;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.concurrent.TimeUnit;

public class TokenCache {

    private static Logger logger = LoggerFactory.getLogger(TokenCache.class);
    public static final String TOKEN_PREFIX = "token_";
	//LoadingCache是附带CacheLoader构建而成的缓存实现。
    private static LoadingCache<String, String> localCache =
            CacheBuilder.newBuilder()
                    .initialCapacity(1000)      //设置缓存的初始化容量
                    .maximumSize(10000)         //缓存的最大容量,超过则移除某些数据
                    .expireAfterAccess(12, TimeUnit.HOURS)  //缓存有效期
                    .build(new CacheLoader<String, String>() {
                        //默认的数据加载实现,当调用get取值的时候,如果key没有对应的值,就调用这
                        //个方法进行加载.
                        @Override
                        public String load(String key) throws Exception {
                            return "null";
                        }
                    });
	//从缓存中存值
    public static void setKey(String key, String value) {
        localCache.put(key, value);
    }
	//从缓存中取值
    public static String getKey(String key) {
        String value = null;
        try {
            value = localCache.get(key);
            if("null".equals(value)) {
                return null;
            }
            return value;
        }catch(Exception e) {
            logger.error("localCache get error", e);
        }
        return null;
    }
}
```

[Guava缓存中文教程]: http://ifeve.com/google-guava-cachesexplained/

#### MD5Util

```java
package com.mmall.util;

import org.springframework.util.StringUtils;
import java.security.MessageDigest;

public class MD5Util {

    private static String byteArrayToHexString(byte b[]) {
        StringBuffer resultSb = new StringBuffer();
        for (int i = 0; i < b.length; i++)
            resultSb.append(byteToHexString(b[i]));

        return resultSb.toString();
    }

    private static String byteToHexString(byte b) {
        int n = b;
        if (n < 0)
            n += 256;
        int d1 = n / 16;
        int d2 = n % 16;
        return hexDigits[d1] + hexDigits[d2];
    }

    /**
     * 返回大写MD5
     *
     * @param origin
     * @param charsetname
     * @return
     */
    private static String MD5Encode(String origin, String charsetname) {
        String resultString = null;
        try {
            resultString = new String(origin);
            MessageDigest md = MessageDigest.getInstance("MD5");
            if (charsetname == null || "".equals(charsetname))
                resultString = byteArrayToHexString(md.digest(resultString.getBytes()));
            else
                resultString = byteArrayToHexString(md.digest(resultString.getBytes(charsetname)));
        } catch (Exception exception) {
        }
        return resultString.toUpperCase();
    }
    
    public static String MD5EncodeUtf8(String origin) {
        //从配置文件读取salt（盐值）
        origin = origin + PropertiesUtil.getProperty("password.salt", "");
        return MD5Encode(origin, "utf-8");
    }
    private static final String hexDigits[] = {"0", "1", "2", "3", "4", "5",
            "6", "7", "8", "9", "a", "b", "c", "d", "e", "f"};

}
```

#### 递归算法的应用

根据一个分类id（categoryId）查找当前节点及所有子节点（categoryId）。

```java
//递归算法,算出子节点
private Set<Category> findChildCategory(Set<Category> categorySet, Integer categoryId) {
    Category category = categoryMapper.selectByPrimaryKey(categoryId);
    if(category != null ) {
        categorySet.add(category);
    }
    //查找子节点,递归算法一定要有一个退出的条件
    List<Category> categoryList = 	categoryMapper.selectCategoryChildrenByParentId(categoryId);	//查找下一级子节点
    for(Category categoryItem : categoryList) {
        //递归调用
        findChildCategory(categorySet, categoryItem.getId());
    }
    return categorySet;
}
```

#### POJO、BO、VO关系

![POJO、BO、VO](C:\Users\华健\Desktop\截图\POJO、BO、VO.PNG)

#### 读取配置文件工具类 PropertiesUtil

```java
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Properties;

public class PropertiesUtil {

    private static Logger logger = LoggerFactory.getLogger(PropertiesUtil.class);
	//声明配置类
    private static Properties props;

    static {
        String filename = "mmall.prpperties";
        try {
            props.load(new InputStreamReader(
                PropertiesUtil.class.getResourceAsStream(filename), "UTF-8"));
        } catch (IOException e) {
            logger.error("读取配置文件失败");
        }
    }

    public static String getProperty(String key) {
        String value = props.getProperty(key.trim());
        if(StringUtils.isBlank(value)) {
            return null;
        }
        return value.trim();
    }
	//配置文件值为空时，返回默认值
    public static String getProperty(String key, String defaultValue) {
        String value = props.getProperty(key.trim());
        if(StringUtils.isBlank(value)) {
            return defaultValue;
        }
        return value.trim();
    }
}
```

#### 时间日期处理工具类(joda time) DateTimeUtil

```java
import org.apache.commons.lang3.StringUtils;
import org.joda.time.DateTime;
import org.joda.time.format.DateTimeFormat;
import org.joda.time.format.DateTimeFormatter;
import java.util.Date;

public class DateTimeUtil {

    private static String STANDARD_FORMAT = "yyyy-MM-dd hh:mm:ss";
    //字符串转日期
    public static Date strToDate(String dateTimeStr, String formatStr) {
        DateTimeFormatter dateTimeFormatter = DateTimeFormat.forPattern(formatStr);
        DateTime dateTime = dateTimeFormatter.parseDateTime(dateTimeStr);
        return dateTime.toDate();
    }
    //日期转字符串
    public static String dateToStr(Date date,String formatStr) {
        if(date == null) {
            return StringUtils.EMPTY;
        }
        DateTime dateTime = new DateTime(date);
        return dateTime.toString(formatStr);
    }
    //字符串转日期，默认格式
    public static Date strToDate(String dateTimeStr) {
        DateTimeFormatter dateTimeFormatter = 
            DateTimeFormat.forPattern(STANDARD_FORMAT);
        DateTime dateTime = dateTimeFormatter.parseDateTime(dateTimeStr);
        return dateTime.toDate();
    }
    //日期转字符串，默认格式
    public static String dateToStr(Date date) {
        if(date == null) {
            return StringUtils.EMPTY;
        }
        DateTime dateTime = new DateTime(date);
        return dateTime.toString(STANDARD_FORMAT);
    }
}
```

#### Mybatis分页插件 Mybatis Helper

相关类：PageHelper、PageInfo

```java
public ServerResponse<PageInfo> getProductList(int pageNum,int pageSize) {
    //1.执行sql前加上这一句
    PageHelper.startPage(pageNum, pageSize);
    PageHelper.orderBy("字段"+" "+"排序方式");	//排序，可选
    //2.执行查询
    List<Product> productList = projectMapper.selectList();
    List<ProductListVo> productListVoList = Lists.newArrayList();
    for(Product productItem : productList) {
        //product——》productListVo
        ProductListVo productListVo = assembleProductListVo(productItem);
        productListVoList.add(productListVo);
    }
    //3.构造PageInfo，直接将查询结果list作为构造参数传入，PageInfo根据List自动封装分页参数
    PageInfo pageResult = new PageInfo(productList);
    //4.（可选）重新设置结果集到PageInfo当中
    pageResult.setList(productListVoList);
    return ServerResponse.createBySuccess(pageResult);
}

```

注：MyBatis分页插件采用AOP的方式，监听sql执行事件，自动为sql加上limit语句，所以sql语句不用写limit。并且sql语句后面不能加分号结尾。

```mysql
<!--查询product List,分页使用Mybatis Helper插件-->
<select id="selectList">
    SELECT
    	<include refid="Base_Column_List"/>
    FROM mmall_product
</select>
```

前台传入两个参数pageNum，pageSize到Controller，可以利用@RequestParam注解的defaultValue属性设置默认值，当前台不传入参数时，使用默认值。

```java
public  ServerResponse<PageInfo> getList(
    @RequestParam(value = "pageNum",defaultValue = "1") int pageNum,
    @RequestParam(value = "pageSize",defaultValue = "10") int pageSize)

	//...                                        
}
```



#### SpringMvc上传文件

配置Bean

```xml
<!-- 文件上传 -->
<bean id="multipartResolver" 	
      class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="maxUploadSize" value="10485760"/> <!-- 10m -->
    <property name="maxInMemorySize" value="4096" />
    <property name="defaultEncoding" value="UTF-8"></property>
</bean>
```

1. ##### Controller层代码

```java
@RequestMapping(value = "upload.do")
@ResponseBody
public ServerResponse upload(@RequestParam(value = "upload_file",required = false) 		
                             MultipartFile file,  
                             HttpServletRequest request, 
                             HttpSession session) {
    User user = (User) session.getAttribute(Const.CURRENT_USER);
    if(user != null) {
        return ServerResponse.createByErrorCodeMessage(
            ResponseCode.NEED_LOGIN.getCode(), "用户未登录,请登录管理员");
    }
    if(iUserService.checkAdminRoel(user).isSuccess()) {
        //要上传的目录路径
        String path = request.getSession().getServletContext().getRealPath("upload");
        //调用service，传入文件和要上传的目录路径
        String targetFileName = iFileService.upload(file, path);
        //上传成功后的图片url
        String url = 		
            PropertiesUtil.getProperty("ftp.server.http.prefix") + targetFileName;
	    //将url和uri返回给前台
        Map fileMap = Maps.newHashMap();
        fileMap.put("uri", targetFileName);
        fileMap.put("url", url);
        return ServerResponse.createBySuccess(fileMap);
    }else {
        return ServerResponse.createByErrorMessage("无权限操作");
    }
}
```

2. ##### Service层代码

```java
@Service("iFileService")
public class FileServiceImpl implements IFileService {
    private static final Logger logger = LoggerFactory.getLogger(FileServiceImpl.class);
    @Override
    public String upload(MultipartFile file, String path) {
        String fileName = file.getOriginalFilename();
        String fileExtensionName = fileName.substring(fileName.lastIndexOf(".") + 1);
        String uploadFileName = UUID.randomUUID().toString() + fileExtensionName;
        logger.info("开始上传文件,上传文件的文件名:{},上传的路径:{},新文件名:{}", 
                    fileName, path, uploadFileName);
        //创建文件夹
        File fileDir = new File(path);
        if(!fileDir.exists()) {
            fileDir.setWritable(true);	//确保tomcat目录可写入
            fileDir.mkdirs();	//mkdirs()：递归创建文件夹
        }
        //创建文件
        File targetFile = new File(path, uploadFileName);
        try {
            //1.上传至tomcat文件夹
            file.transferTo(targetFile);
            //2.将文件上传至FTP服务器，调用FTPUtil工具类上传
            FTPUtil.uploadFile(Lists.newArrayList(targetFile));
            //3.删除tomcat中的文件
            targetFile.delete();
        } catch (IOException e) {
            logger.error("文件上传异常", e);
            e.printStackTrace();
        }
        return targetFile.getName();
    }
}
```

3. ##### FTP上传工具类（FTPUtil）

```java
public class FTPUtil {
    private static final Logger logger = LoggerFactory.getLogger(FTPUtil.class);
    //从配置文件中读取FTP服务器信息
    private static String ftpIp = PropertiesUtil.getProperty("ftp.server.ip");
    private static String ftpUser = PropertiesUtil.getProperty("ftp.user");
    private static String ftpPass = PropertiesUtil.getProperty("ftp.pass");
    //构造器
    public FTPUtil(String ftpIp, Integer ftpPort, String ftpUser, String ftpPass) {
        this.ip = ftpIp;
        this.port = ftpPort;
        this.user = ftpUser;
        this.pass = ftpPass;
    }
    //暴露给外部的上传文件接口
    public static boolean uploadFile(List<File> fileList) throws IOException{
        FTPUtil ftpUtil = new FTPUtil(ftpIp, 21, ftpUser, ftpPass);
        logger.info("开始连接ftp服务器");
        boolean result = ftpUtil.uploadFile("img", fileList);
        logger.info("开始连接ftp服务器,结束上传,上传结果:{}");
        return result;
    }
    //上传文件处理逻辑
    private boolean uploadFile(String remotePath, List<File> fileList) 
        										throws IOException {
        boolean uploaded = false;
        FileInputStream fis = null;
        if(connectServer(this.ip, this.port, this.user, this.pass)) {
            try {
                ftpClient.changeWorkingDirectory(remotePath);
                ftpClient.setBufferSize(1024);
                ftpClient.setFileType(FTPClient.BINARY_FILE_TYPE);
                ftpClient.enterLocalPassiveMode();
                for (File fileItem : fileList) {
                    fis = new FileInputStream(fileItem);
                    ftpClient.storeFile(fileItem.getName(), fis);
                }
            }catch (IOException e){
                logger.error("上传文件异常", e);
                uploaded = false;
                e.printStackTrace();
            }finally {
                fis.close();
                ftpClient.disconnect();
            }
        }
        return uploaded;
    }
    //连接FTP服务器
    private boolean connectServer(String ftpIp, Integer ftpPort, 
                                  String ftpUser, String ftpPass) {
        boolean isSuccess = false;
        ftpClient = new FTPClient();
        try {
            //1.连接ftp
            ftpClient.connect(ftpIp);
            //2.登录
            isSuccess = ftpClient.login(ftpUser, ftpPass);
        } catch (IOException e) {
            logger.error("连接FTP服务器异常",e);
        }
        return isSuccess;
    }

    private String ip;
    private Integer port;
    private String user;
    private String pass;
    private FTPClient ftpClient;
	//setter、setter
}
```

富文本上传文件（simditor）

富文本中对于返回值有自己的要求,我们使用是simditor所以按照simditor的要求进行返回。

```json
{
    "success": true/false,
    "msg": "error message", # optional
    "file_path": "[real file path]"
}
```



```java
@RequestMapping(value = "richtext_img_upload.do")
@ResponseBody
public Map richtextImgUpload(@RequestParam(value = "upload_file",required = false) 
                             MultipartFile file, HttpSession session,
                             HttpServletRequest request, HttpServletResponse response) {
    Map resultMap = Maps.newHashMap();
    User user = (User) session.getAttribute(Const.CURRENT_USER);
    if(user != null) {
        resultMap.put("success",false);
        resultMap.put("msg","请登录管理员");
        return resultMap;
    }
    if(iUserService.checkAdminRoel(user).isSuccess()) {
        //上传目标目录
        String path = request.getSession().getServletContext().getRealPath("upload");
        String targetFileName = iFileService.upload(file, path);
        if(StringUtils.isBlank(targetFileName)) {
            resultMap.put("success",false);
            resultMap.put("msg","上传失败");
            return resultMap;
        }
        String url = 
            PropertiesUtil.getProperty("ftp.server.http.prefix")+targetFileName;

        resultMap.put("success",true);
        resultMap.put("msg","上传成功");
        resultMap.put("file_path",url);
        response.setHeader("Access-Control-Allow-Headers", "X-File-Name");
        return resultMap;
    }else {
        resultMap.put("success",false);
        resultMap.put("msg","无权限操作");
        return resultMap;
    }
}
```

#### MyBatis的<foreach>标签、<where>标签及<if>标签使用方法

```xml
<!--根据产品名和分类id模糊搜索产品-->
<select id="selectByNameAndCategoryIds" resultMap="BaseResultMap" parameterType="map">
    SELECT
    	<include refid="Base_Column_List"/>
    FROM mmall_product
    <where>
        <if test="productName != null">
            and name like #{productName}
        </if>
        <if test="categoryIdList != null">
            and category_id in
            <foreach item="item" index="index" open="(" separator="," close=")" 
                     							collection="categoryIdList">
                #{item}
            </foreach>
        </if>
    </where>
</select>
```

注： 

1. <where>标签会自动处理<if>标签中的and关键字，即if标签有不为空的，紧跟着where后面的第一个条件的and会自动去除。
2. <foreach>标签遍历集合时可以配置open、separate和close。


#### BigDecimal工具类

在进行带有小数的数学运算的时候，需要用到float、double等类型，会出现丢失精度的问题。

使用BigDecimal的String构造器会完美解决这个问题。

封装的BigDecimal工具类如下：

```java
public class BigDecimalUtil {

    private BigDecimalUtil() {}

    public static BigDecimal add(double v1, double v2) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.add(b2);
    }

    public static BigDecimal substract(double v1, double v2) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.subtract(b2);
    }

    public static BigDecimal multiply(double v1, double v2) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.multiply(b2);
    }

    public static BigDecimal devide(double v1, double v2) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.divide(b2, 2, BigDecimal.ROUND_HALF_UP);
    }
}
```

### Spring Security

#### 初始化

1. 新建一个带有@Configuration注解的配置类
2. 继承WebSecurityConfigurerAdapter（浏览器模块）
3. 重写configure(HttpSecurity http)方法
4. 开启表单登录验证

```java
@Configuration
public class BrowserSecurityConfig extends WebSecurityConfigurerAdapter {
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.formLogin()	//开启表单验证
			.and()
			.authorizeRequests()
			.anyRequest()
			.authenticated();
	}
}
```

#### Spring Securit原理

Spring Security本质上是一个过滤器链，如图：

![Spring Security基本原理](C:\Users\华健\Desktop\截图\SpringSecurity基本原理.PNG)

1. SecurityContextPersistenceFilter：第一个过滤器，请求进来时，检查session中是否有认证信息（Authentication），如果有，将authentication放到（SecurityContextHolder）线程中，跳过认证流程；响应返回时，将认证成功的认证信息放到session中，使得认证结果在多个请求之间共享。	
2. 绿色过滤器是认证过滤器，通过代码进行配置，是可选的，根据认证信息的类型只会选择则其中一个过滤器进行认证。
3. RememberMeAuthenticationFilter：记住我过滤器，绿色过滤器链中的倒数第二个，当前面的过滤器都没法认证的时候，会尝试调用该过滤器。
4. AnonymousAuthenticationFilter：处于绿色的过滤的最后一个，判断SecurityContext是否有Authentication，如果没有，封装一个匿名的Token。
5. FilterSecurityInteceptor：最后一个过滤器，对请求的认证、权限进行判断，如果都满足则方形，请求能访问到服务；如果请求需要进行认证而为认证或者没有权限进行访问等，将会抛出具体的异常给倒数第二个过滤器。
6. ExceptionTranstationFilter：接收FilterSecurityInteceptor抛出的异常，进行相应的处理。

绿色过滤器是通过配置类中的代码控制的，其它的过滤器不可控制。

#### 用户认证流程

![Spring Security认证流程](C:\Users\华健\Desktop\截图\SpringSecurity认证流程.PNG)

#### 自定义用户认证逻辑

1. 自定义一个认证类，实现**UserDetailsService接口**。
2. 重写**loadUserByUsername( )方法**，方法里是根据用户名查询用户的逻辑，一般是从数据库中获取用户。
3. 返回User对象，这个User类必须实现**UserDetails接口**，Spring Security会拿登录信息与查询出的对象进行校验，检查登陆信息是否正确。
4. UserDetails接口的实现类封装用户的校验逻辑，根据业务需求重写接口的7个方法。

UserDetails接口：

```java
public interface UserDetails extends Serializable {
	//获取用户的权限集合
	Collection<? extends GrantedAuthority> getAuthorities();
	//获取用户的密码
	String getPassword();
	//获取用户的用户名
	String getUsername();
	//判断账户是否过期
	boolean isAccountNonExpired();
	//判断账户是否被锁
	boolean isAccountNonLocked();
	//判断账户密码是否过期
	boolean isCredentialsNonExpired();
	//判断账户是否已删除（逻辑删除）
	boolean isEnabled();
}
```

示例：

```java
public class MyUserDetailsService implements UserDetailsService {
	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        //根据用户名查询用户
        //....
        //返回用户，该User实现了UserDetailsService，构造器的七个参数分别是用户名、密码和权限集合、
        //账户未过期、账户未被锁定、密码未过期、账户可用
		return new User(username, "123456", 
                        AuthorityUtils.commaSeparatedStringToAuthorityList("admin"),
                        true,true,true,true);
	}
}
```

自定义的认证流程：

```java
@Configuration
public class BrowserSecurityConfig extends WebSecurityConfigurerAdapter {

	@Autowired
	private SecurityProperties securityProperties;
	@Autowired
	private ImoocAuthenticationSuccessHandler imoocAuthenticationSuccessHandler;
	@Autowired
	private ImoocAuthenctiationFailureHandler imoocAuthenctiationFailureHandler;
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.formLogin()
				.loginPage("/authentication/require")			    //自定义的的未登录处理url
				.loginProcessingUrl("/authentication/form")		    //自定义的登录表单提交url
				.successHandler(imoocAuthenticationSuccessHandler)	//自定义的认证成功处理器
				.failureHandler(imoocAuthenctiationFailureHandler)	//自定义的认证失败处理器
			.and()
			.authorizeRequests()
				.antMatchers(securityProperties.getBrowserProperties().getLoginPage(),
                         "/authentication/require").permitAll()
				.anyRequest().authenticated()
			.and()
			.csrf().disable();	
	}	
}
```

#### 记住我

原理

![](C:\Users\华健\Desktop\截图\记住我原理.PNG)

使用

1. 页面上加入记住我单选框，name="remember-me"

   ```html
   <td colspan='2'><input name="remember-me" type="checkbox" value="true" />记住我</td>
   ```

2. Java配置

   ```java
   @Autowired
   private DataSource dataSource;
   @Autowired
   private UserDetailsService userDetailsService;

   //记住我token仓库配置
   @Bean
   public PersistentTokenRepository persistentTokenRepository() {
       JdbcTokenRepositoryImpl tokenRepository = new JdbcTokenRepositoryImpl();
       tokenRepository.setDataSource(dataSource);
       tokenRepository.setCreateTableOnStartup(true);	//启动时自动创建表
       return tokenRepository;
   }

   @Override
   protected void configure(HttpSecurity http) throws Exception {
       http.formLogin()
           	.loginPage("/authentication/require")
           	.loginProcessingUrl("/authentication/form")
           	.successHandler(imoocAuthenticationSuccessHandler)
           	.failureHandler(imoocAuthenctiationFailureHandler)
           .and()
           .rememberMe()	//记住我
           	.tokenRepository(persistentTokenRepository())	//token仓库
           	.tokenValiditySeconds(1000)		//过期时间
           	.userDetailsService(userDetailsService)	 //拿到token调用userDetailsService进行登录
           .and()
           .csrf().disable();	

       authorizeConfigManager.config(http.authorizeRequests());
   }
   ```

#### Session管理

**session概念：**

​	在WEB开发中，服务器可以为每个用户浏览器创建一个会话对象（session对象），注意：一个浏览器独占一个session对象(默认情况下)。因此，在需要保存用户数据时，服务器程序可以把用户数据写到用户浏览器独占的session中，当用户使用浏览器访问其它程序时，其它程序可以从用户的session中取出该用户的数据，为用户服务。

[Session简介]: https://www.cnblogs.com/xdp-gacl/p/3855702.html

**session与cookie的区别：**

```markdown
- Cookie是把用户的数据写给用户的浏览器。
- Session技术把用户的数据写到用户独占的session中。
- Session对象由服务器创建，开发人员可以调用request对象的getSession方法得到session对象
```

1. 在application.properties里配置session过期时间，单位秒

```properties
#session默认过期时间是60秒，即使配置小于60，SpringBoot也会默认置为60
server.session.timeout=50
```

2. 在安全配置文件里进行session管理

```java
.sessionManagement()
    .invalidSessionUrl("/session/invalid")	//session失效跳转url
    .maximumSessions(1)		//当前用户最大session数
    .maxSessionsPreventsLogin(true)	//当前session还存在时，阻止另外一个浏览器登录
```

### Spring Security控制授权

#### 授权的简单使用

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.formLogin()
        .loginPage("/authentication/require")
        .loginProcessingUrl("/authentication/form")
        .successHandler(imoocAuthenticationSuccessHandler)
        .failureHandler(imoocAuthenctiationFailureHandler)
        .and()
        .authorizeRequests()
        .antMatchers(securityProperties.getBrowserProperties().getLoginPage(),
        			"/authentication/require").permitAll()
        .antMatchers(HttpMethod.GET,"/user/*").hasRole("ADMIN")	//指定/user/*的GET请求需要ADMIN权限
        .anyRequest()
        .authenticated()
        .and()
        .csrf().disable();	
}

//UserDetailsService实现类
@Component
public class MyUserDetailsService implements UserDetailsService {
	private Logger logger = LoggerFactory.getLogger(MyUserDetailsService.class);
	
	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		logger.info("登录的用户是：" + username);
		return new User(username, "123456", 
                        AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_ADMIN"));
        				//第三个参数为用户的权限，这里是ADMIN角色
	}

}
```

#### Spring Security授权流程分析

![](C:\Users\华健\Desktop\截图\SpringSecurity授权流程分析.PNG)

#### Spring Security权限表达式

![](C:\Users\华健\Desktop\截图\权限表达式.PNG)

如果需要同时使用多个权限表达式，需要使用access( )表达式，如下：

```java
.antMatchers(HttpMethod.GET,"/user/*").access("hasRole('ADMIN' ) and hasIpAddress('xxxx')")
```

#### 静态权限的配置

因为核心的安全模块需要被其它模块引用，里面只能进行基础的安全配置，如登录、错误处理等。具体的涉及到业务的权限配置需要在各自的模块当中进行配置。

![](C:\Users\华健\Desktop\截图\多模块权限思路.PNG)

1. 在核心模块定义配置提供器（AuthorizeConfigProvider）接口，具体的引用模块提供实现类，在config( )方法里进行模块的授权配置。

```java
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configurers.ExpressionUrlAuthorizationConfigurer;
public interface AuthorizeConfigProvider {

	void config(
        ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry config);
}
```

2. 核心安全模块的实现类：CoreAuthorizeConfigProvider

```java
@Component
public class CoreAuthorizeConfigProvider implements AuthorizeConfigProvider {

	@Autowired
	private SecurityProperties securityProperties;
	
	@Override
	public void config(
       ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry config) {
		
        config.antMatchers(
            securityProperties.getBrowserProperties().getLoginPage(),"/authentication/require")
			.permitAll();
				
	}
}
```

Demo模块的AuthorizeConfigProvider实现类：DemoAuthorizeConfigProvider

```java
@Component
public class DemoAuthorizeConfigProvider implements AuthorizeConfigProvider {

	@Override
	public void config(
       ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry config) {
		
		config.antMatchers(HttpMethod.GET, "/user/*").hasRole("ADMIN");	
	}
}
```

3. 声明授权配置管理器（AuthorizeConfigManager）接口，并提供一个实现类，其作用是遍历Spring容器中的说有Provider实现类，即搜集涉及到的模块所有的权限配置。

```java
public interface AuthorizeConfigManager {

	void config(
        ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry config);
}

```

```java
@Component
public class CoreAuthorizeConfigManager implements AuthorizeConfigManager {

	@Autowired
	private Set<AuthorizeConfigProvider> authorizeConfigProviders;
	
	@Override
	public void config(
       ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry config) {
		//遍历所有配置提供器
		for(AuthorizeConfigProvider authorizeConfigProvider : authorizeConfigProviders) {
			authorizeConfigProvider.config(config);
		}
		//其它所有请求需要认证
		config.anyRequest().authenticated();
	}
}
```

4. 在使用的模块的安全配置文件中注入授权配置管理器（CoreAuthorizeConfigManager），删除掉原来的授权配置。

```java
@Autowired
private AuthorizeConfigManager authorizeConfigManager;

@Override
protected void configure(HttpSecurity http) throws Exception {
    http.formLogin()
        .loginPage("/authentication/require")
        .loginProcessingUrl("/authentication/form")
        .successHandler(imoocAuthenticationSuccessHandler)
        .failureHandler(imoocAuthenctiationFailureHandler)
        .and()
        .csrf().disable();	
	
    //引入所有配置
    authorizeConfigManager.config(http.authorizeRequests());
}
```

#### 业务系统和内管系统

![业务系统和内管系统](C:\Users\华健\Desktop\截图\业务系统和内管系统.PNG)

#### 通用RBAC（Role-Based Access Control）数据模型

![](C:\Users\华健\Desktop\截图\RBAC.PNG)

动态权限的配置

静态的权限配置是写死在代码里的，不在符合需求，这里使用RBAC数据模型，需要自己定义一个符合业务需求的权限表达式，从数据库中查询当前用户拥有权限的所有url，匹配上权限表达式就返回true。

```java
//自定义的权限表达式
@Component("rbacService")
public class RbacServiceImpl implements RbacService {

	private Logger logger = LoggerFactory.getLogger(RbacServiceImpl.class);
	private AntPathMatcher antPathMatcher = new AntPathMatcher();
	
    //第一个参数：当前请求
    //第二个参数：认证信息
	@Override
	public boolean hasPremission(HttpServletRequest request, Authentication authentication) {
		
		boolean hasPremission = false;
		//获取当前用户的信息
		Object principal = authentication.getPrincipal();
		if(principal instanceof UserDetails) {
			//获取用户名
			String username = ((UserDetails) principal).getUsername();
			logger.info("从数据库中查询用户" + username + "拥有权限的url");
			//根据用户名获取该用户用户权限的url集合,一旦与请求的url匹配，返回true
			List<String> urls = new ArrayList<String>();
			for(String url : urls) {
				if(antPathMatcher.match(url, request.getRequestURI())) {
					hasPremission = true;
					break;
				}
			}	
		}
		return hasPremission;
	}
}
```

自定义的权限表达式的使用：.access("@rbacService.hasPremission(request, authentication)")

```java
@Component
@Order(Integer.MAX_VALUE)
public class DemoAuthorizeConfigProvider implements AuthorizeConfigProvider {

	@Override
	public void config(
       ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry config) {
		//config.antMatchers(HttpMethod.GET, "/user/*").hasRole("admin");	
		config.anyRequest().access("@rbacService.hasPremission(request, authentication)");
	}
}
```

注：**@Order(Integer.MAX_VALUE)**：**AuthorizeManager**在搜集所有**AuthorizeProvider**的时候，会按括号里的整数值从小到大的放入集合中。这里使用 Integer.MAX_VALUE 是因为 **anyRequest( )** 需要放在所有配置的最后一局，不然会覆盖前面的配置。

### Tomcat集群

#### Tomcat集群能带来什么

- 提高服务器的性能，并发能力，以及高可用性
- 提供项目架构的横向扩展能力

#### Tomcat集群实现原理

通过Nginx负载均衡进行请求转发

#### 一期架构与二期架构对比

1. 一期架构

   ![](C:\Users\华健\Desktop\截图\一期架构.PNG)

2. 二期架构

   ![](C:\Users\华健\Desktop\截图\二期架构.PNG)

3. 二期真架构

   ![](C:\Users\华健\Desktop\截图\二期真架构.PNG)

####   Nginx负载均衡配置 、常用策略 、场景及特点

- 轮询

  - 优点：实现简单
  - 缺点：不考虑每台服务器处理能力

- 权重

  - 优点：考虑了每台服务器处理能力的不同

- ip hash

  - 优点：能实现同一个用户访问同一个服务器
  - 缺点：根据ip hash不一定平均

- url hash（第三方）

  - 优点：能实现同一个服务访问同一个服务器
  - 缺点：根据url hash分配请求会不平均，请求频繁的url请求到同一个服务器上

- fair（第三方）

  - 特点：按后端服务器的响应时间来分配请求，响应时间短的优先分配

- 负载均衡参数扩展知识点

  ![](C:\Users\华健\Desktop\截图\负载均衡参数.PNG)

#### Linux单击配置多个Tomcat实例

1. 下载压缩包，分别解压到tomcat1、tomcat2两个目录下

2. 配置环境变量

   ```bash
   export CATALINA_BASE=/usr/local/tomcat1
   export CATALINA_HOME=/usr/local/tomcat1
   export TOMCAT_HOME=/usr/local/tomcat1
   
   export CATALINA_2_BASE=/usr/local/tomcat2
   export CATALINA_2_HOME=/usr/local/tomcat2
   export TOMCAT_2_HOME=/usr/local/tomcat2
   ```

3. tomcat1不需要修改，修改tomcat2的bin目录下的catalina.sh

   ```bash
   # OS specific support.  $var _must_ be set to either true or false.
   # 增加语句
   export CATALINA_BASE=$CATALINA_2_BASE
   export CATALINA_HOME=$CATALINA_2_HOME
   ```

4. 修改tomcat2的bin/server.xml中的端口

   ```xml
   <!-- 第一处 ：8005改为9005-->
   <Server port="9005" shutdown="SHUTDOWN"></Server>
   <!-- 第二处 ：8080改为9080-->
   <Connector port="9080" protocol="HTTP/1.1" connectionTimeout="20000" 
              redirectPort="8443 URIEncoding="UTF-8" />
   <!-- 第三处 ：8009改为9009-->
   <Connector port="9009" protocol="AJP/1.3" redirectPort="8443" />
   ```

5. Nginx负载均衡权重策略配置

   ```conf
   upstream www.imooc.com{
   	server 127.0.0.1:8080 weight=1;
   	server 127.0.0.1:9080 weight=2;
   	#server www.imooc.com:8080;
   	#server www.imooc.com:9080;
   }
   
   server {  
   	listen 80; 
   	autoindex on; 
   	server_name www.imooc.com; 
   	access_log c:/access.log combined; 
   	index index.html index.htm index.jsp index.php; 
   	#error_page 404 /404.html; 
   	if ( $query_string ~* ".*[\;'\<\>].*" ){ 
   		return 404; 
   	} 
   	location / { 
   		proxy_pass http://www.imooc.com;
   		add_header Access-Control-Allow-Origin *; 
   	} 
   }
   ```

   

### Spring Session

1. 引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.session</groupId>
       <artifactId>spring-session-data-redis</artifactId>
       <version>1.2.0.RELEASE</version>
   </dependency>
   ```

2. 创建applicationContext-spring-session.xml配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
          xmlns:tx="http://www.springframework.org/schema/tx" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
   
   
       <bean id="redisHttpSessionConfiguration" class="org.springframework.session.data.redis.config.annotation.web.http.RedisHttpSessionConfiguration">
           <!--session时长：默认1800s-->
           <property name="maxInactiveIntervalInSeconds" value="1800" />
       </bean>
   
       <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
           <property name="maxTotal" value="20"/>
       </bean>
   
       <bean id="jedisConnectionFactory" 	class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
           <property name="hostName" value="192.168.0.8"/>
           <property name="port" value="6379"/>
           <property name="poolConfig" ref="jedisPoolConfig"/>
       </bean>
   
   </beans>
   ```

   

### MySql开发技巧

SQL语句类型

![](C:\Users\华健\Pictures\SQL语句类型.PNG)