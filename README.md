<font size=4.5>

**Sping Cloud Alibaba Nacos**

---
#### 1.什么是Nacos？

> [Nacos](https://github.com/alibaba/Nacos) 是阿里巴巴开源的一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

#### 2.Nacos Config整合

> Nacos Config Starter实现Spring Cloud应用程序的外部化配置。

##### 2.1 启动 Nacos Server 并添加配置

> 1.下载地址：
>
> 1. 直接下载：[Nacos Server 下载页](https://github.com/alibaba/nacos/releases)
> 2. 源码构建：[Github 项目页面](https://github.com/alibaba/nacos)
>
>
> 2.启动
>
> 1. Linux/Unix/Mac 操作系统，执行命令(默认集群版本启动)
>   `sh startup.sh -m standalone`(单机) `sh startup.sh -m cluster`(集群)
> 2. Windows 操作系统，执行命令 `cmd startup.cmd`
> 3. 默认账户密码：nacos/naocs
>
>
> 3.在命令行执行如下命令，向 Nacos Server 中添加一条配置，也可在控制台中手动添加。
>
>     curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacosConfig.properties&group=DEFAULT_GROUP&content=user.name=Mr.Kong%0Auser.age=24"
>        
>     添加的配置的详情如下
>        
>      dataId 为 nacosConfig.properties
>      group 为 DEFAULT_GROUP
>      
>      内容如下:
>      
>      	user.name=Mr.Kong
>      	user.age=24
>
> ![](https://gitee.com/FocusProgram/PicGo/raw/master/20200305152113.png)

##### 2.1 引入pom文件

```
 <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
 </dependency>
```

##### 2.2 添加bootstrap.properties

```
spring.application.name=nacosConfig  #保持跟Nacos的id名一致
spring.cloud.nacos.config.server-addr=ip:8848 #服务器ip或本地ip
```

##### 2.3 获取相应的配置

```java
@RefreshScope //打开动态刷新功能
 class SampleController {

 	@Value("${user.name}")
 	String userName;

 	@Value("${user.age}")
 	int age;
 }
```

##### 2.4 应用启动

> 1.添加/application.properties
>
>  server.port=9000
>  management.endpoints.web.exposure.include=*
>
> 2.App启动
>
> ![](https://gitee.com/FocusProgram/PicGo/raw/master/20200305153453.png)

##### 2.5 原理

> #### 1.Nacos Config 数据结构
>
> Nacos Config 主要通过 dataId 和 group 来唯一确定一条配置.
>
> Nacos Client 从 Nacos Server 端获取数据时，调用的是此接口 `ConfigService.getConfig(String dataId, String group, long timeoutMs)`
>
> #### 2.Spring Cloud 应用获取数据
>
> ##### dataId
>
> 在 Nacos Config Starter 中，dataId 的拼接格式如下
>
>  ${prefix} - ${spring.profiles.active} . ${file-extension}
>
> - `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
>
> - `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)
>
> -  注意:当 activeprofile 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成 prefix.prefix
>
> - `file-extension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension`来配置。 目前只支持 `properties` 类型。
>
> 
>
> ##### group
>
> - `group` 默认为 `DEFAULT_GROUP`，可以通过 `spring.cloud.nacos.config.group` 配置。
>
>
> #### 3.自动注入
>
> Nacos Config Starter 实现了 `org.springframework.cloud.bootstrap.config.PropertySourceLocator`接口，并将优先级设置成了最高。
>
> 在 Spring Cloud 应用启动阶段，会主动从 Nacos Server 端获取对应的数据，并将获取到的数据转换成 PropertySource 且注入到 Environment 的 PropertySources 属性中，所以使用 @Value 注解也能直接获取 Nacos Server 端配置的内容。
>
> #### 4.动态刷新
>
> Nacos Config Starter 默认为所有获取数据成功的 Nacos 的配置项添加了监听功能，在监听到服务端配置发生变化时会实时触发 `org.springframework.cloud.context.refresh.ContextRefresher` 的 refresh 方法 。
>
> 如果需要对 Bean 进行动态刷新，给类添加 `@RefreshScope` 或 `@ConfigurationProperties`注解。
>
> #### 5.Endpoint 信息查看
>
> Springboot支持这一点，Nacos Config也同时可以使用Endpoint来暴露信息。
>
> 在maven 中添加 `spring-boot-starter-actuator`依赖，并在配置中允许 Endpoints 的访问。
>
> - Spring Boot 1.x 中添加配置 management.security.enabled=false
> - Spring Boot 2.x 中添加配置 management.endpoints.web.exposure.include=*
>
>
> Spring Boot 1.x 可以通过访问 [http://127.0.0.1:9000/nacos_config](http://127.0.0.1:9000/nacos_config) 来查看 Nacos Endpoint 的信息。
>
> Spring Boot 2.x 可以通过访问 [http://127.0.0.1:9000/actuator/nacos-config](http://127.0.0.1:9000/actuator/nacos-config) 来访问。
> ![](https://gitee.com/FocusProgram/PicGo/raw/master/20200305162410.png)

##### 2.6 更多

| 配置项             | key                                            | 默认值            | 说明                                       |
|-----------------|------------------------------------------------|----------------|------------------------------------------|
| 服务端地址           | spring\.cloud\.nacos\.config\.server\-addr     |                |                                          |
| DataId前缀        | spring\.cloud\.nacos\.config\.prefix           |                | spring\.application\.name                |
| Group           | spring\.cloud\.nacos\.config\.group            | DEFAULT\_GROUP |                                          |
| dataID后缀及内容文件格式 | spring\.cloud\.nacos\.config\.file\-extension  | properties     | dataId的后缀，同时也是配置内容的文件格式，目前只支持 properties |
| 配置内容的编码方式       | spring\.cloud\.nacos\.config\.encode           | UTF\-8         | 配置的编码                                    |
| 获取配置的超时时间       | spring\.cloud\.nacos\.config\.timeout          | 3000           | 单位为 ms                                   |
| 配置的命名空间         | spring\.cloud\.nacos\.config\.namespace        |                | 常用场景之一是不同环境的配置的区分隔离，例如开发测试环境和生产环境的资源隔离等。 |
| AccessKey       | spring\.cloud\.nacos\.config\.access\-key      |                |                                          |
| SecretKey       | spring\.cloud\.nacos\.config\.secret\-key      |                |                                          |
| 相对路径            | spring\.cloud\.nacos\.config\.context\-path    |                | 服务端 API 的相对路径                            |
| 接入点             | spring\.cloud\.nacos\.config\.endpoint         | UTF\-8         | 地域的某个服务的入口域名，通过此域名可以动态地拿到服务端地址           |
| 是否开启监听和自动刷新     | spring\.cloud\.nacos\.config\.refresh\.enabled | TRUE           |                                          |


#### 3.Nacos Discovery 整合

> Nacos Discovery Starter 完成 Spring Cloud 应用的服务注册与发现。

##### 3.1 添加依赖

```
 <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
 </dependency>

```

##### 3.2 application.properties

```
spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
```

##### 3.3 服务注册和发现

```java
@SpringBootApplication
@EnableDiscoveryClient
public class ProviderApplication {

 	public static void main(String[] args) {
 		SpringApplication.run(Application.class, args);
 	}

 	@RestController
 	class EchoController {
 		@RequestMapping(value = "/echo/{string}", method = RequestMethod.GET)
 		public String echo(@PathVariable String string) {
 				return string;
 		}
 	}
 }
```

##### 3.4 启动Nacos服务端

> 1. 通过从源代码下载或构建来安装Nacos Server。**推荐的最新版本Nacos Server**
> 1. 下载Nacos Server [下载页面](https://github.com/alibaba/nacos/releases)
> 2. 从源代码构建：通过git clone [git@github.com](mailto:git@github.com)获取源代码：来自Github Nacos的alibaba / Nacos.git并构建您的代码。有关详细信息，请参阅[构建](https://nacos.io/en-us/docs/quick-start.html)
>
>
> 2. 运行
>
> 1. Linux / Unix / Mac，执行 `sh startup.sh -m standalone`
> 2. Windows，执行 `cmd startup.cmd`

##### 3.5 启动客户端

> 1.添加application.properties配置
>
>   spring.application.name=service-provider
>   server.port=8070


##### 3.6 查询服务

![](https://gitee.com/FocusProgram/PicGo/raw/master/20200305163858.png)

##### 3.7 服务发现

> Nacos Discovery Starter默认集成了Ribbon，因此对于使用Ribbon进行负载平衡的组件，您> 可以直接使用Nacos服务发现。
>
> NacosServerList实现com.netflix.loadbalancer.ServerList接口并在@ConditionOnMissing> Bean下自动注入它。

##### 3.8 原理

> #### 服务注册
>
> 在Spring云应用程序的启动阶段，将监视WebServerInitializedEvent事件。在初始化Web容器后收到WebServerInitializedEvent事件时，将触发注册操作，并调用ServiceRegistry注册方法以将服务注册到Nacos Server。
>
> #### 服务发现
>
> NacosServerList实现com.netflix.loadbalancer.ServerList接口并在@ConditionOnMissingBean下自动注入它。功能区默认为集成。
>
> ### 端点
>
> Nacos Discovery Starter还支持Spring Boot执行器端点的实现。
>
> **先决条件：**
>
> 将依赖spring-boot-starter-actuator添加到pom.xml文件中，并配置端点安全策略。
>
> Spring Boot 1.x：添加配置management.security.enabled = false
>
> Spring Boot 2.x：添加配置management.endpoints.web.exposure.include = *
>
> 要查看端点信息，请访问以下URL：
>
> Spring Boot1.x：Nacos Discovery端点URL是[http://127.0.0.1:8070/nacos_discovery](http://127.0.0.1:8070/nacos_discovery)。
>
> Spring Boot2.x：Nacos Discovery端点URL是[http://127.0.0.1:8070/actuator/nacos-discovery](http://127.0.0.1:8070/actuator/nacos-discovery)。
>
> ![](https://gitee.com/FocusProgram/PicGo/raw/master/20200305165037.png)

##### 3.9 更多
| 配置项       | 键                                                   | 默认值                       | 描述                                                  |
|-----------|-----------------------------------------------------|---------------------------|-----------------------------------------------------|
| 服务器地址     | spring\.cloud\.nacos\.discovery\.server\-addr       |                           | nacos注册中心地址                                         |
| 服务名       | spring\.cloud\.nacos\.discovery\.service            | spring\.application\.name | 服务名                                                 |
| 权重        | spring\.cloud\.nacos\.discovery\.weight             | 1                         | 值从1到100，值越大，重量越大                                    |
| IP        | spring\.cloud\.nacos\.discovery\.ip                 |                           | ip address to registry，最高优先级                        |
| 网络接口      | spring\.cloud\.nacos\.discovery\.network\-interface |                           | 未配置IP时，注册的IP地址为网络接口对应的IP地址。如果未配置此项，则默认采用第一个网络接口的地址。 |
| 端口        | spring\.cloud\.nacos\.discovery\.port               | \-1                       | 注册端口，无需配置即可自动检测                                     |
| namesapce | spring\.cloud\.nacos\.discovery\.namespace          |                           | 开发环境（dev、pro等）                                      |
| accesskey | spring\.cloud\.nacos\.discovery\.access\-key        |                           |                                                     |
| secretkey | spring\.cloud\.nacos\.discovery\.secret\-key        |                           |                                                     |
| 元数据       | spring\.cloud\.nacos\.discovery\.metadata           |                           | 扩展数据，使用Map格式配置                                      |
| 日志名称      | spring\.cloud\.nacos\.discovery\.log\-name          |                           |                                                     |
| 端点        | spring\.cloud\.nacos\.discovery\.endpoint           |                           | 服务的域名，通过该域名可以动态获取服务器地址。                             |
| 集成功能区     | ribbon\.nacos\.enabled                              | TRUE                      |                                                     |

</font>