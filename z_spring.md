# spring

#### 简单笔记
@SpringBootApplication注解，只要将main类放到root package中

多环境配置
application.properties可以配置开发生产环境

Mbytes
数据库操作工具

异常统一处理
RestControllerAdvice ：控制层通知器，这里用于统一拦截异常，进行响应处理
在控制层（controller）抛出异常，RestControllerAdvice注解可以统一接收到并实现处理逻辑并返回。

Freemarker

Restful 
@RestController：表示一个支持Restful的控制器
RequestBody  接受json字符串，

Redis 
NoSQL：泛指非关系型的数据库，支持键值(Key-Value)存储数据库～～
序列化
缓存
命令：
redis-server /usr/local/etc/redis.conf

第一课：
application 配置文件
可以做的事：端口，URL前缀，自定义属性、配置

第二课：
ResetController
RequestMapping, GetMapping ,直接对象

请求参数获取：1、PathVariale ：url中的参数 2、RequestParam 请求参数的值 
数据库
spa
更新操作：新建对象，保存久OK
查询：
事务：transationl 一系列，要么都成功，要么都失败，不存在部分成功。

第三节课：
表单验证：@valid BindingResult @min
App 日志拦截 @aspect
统一异常处理
单元测试@runwith @SpringBootTest
IDEA 快捷键构建测试类


#### 详记
Spring Boot:
Spring平台及第三方库:开箱即用
Servlet容器: 运行代码环境，Tomcat，Jetty（网页）
Spring Boot CLI:一个命令行工具
Application：一个main函数作为主入口，具体run方法会启动嵌入式的Tomcat并初始化Spring环境及其各Spring组件



后台结构：
Model = 模型 ：持久化的数据一一对应，如Mysql和MongoDB。
DAO = Data Access Object = 数据存取对象 ：底层数据库通信，负责对数据库的增删改查。
Service = 服务
ServiceImpl = 服务真实实现
Controller = 控制中心，所有的指令，调度都从这里发出去。
Util = 工具





application.properties
自定义属性
多环境配置
application-dev.properties：开发环境
application-prod.properties：生产环境
spring.profiles.active=dev

mybatis.typeAliasesPackage 配置为 org.spring.springboot.domain，指向实体类包路径。
mybatis.mapperLocations 配置为 classpath 路径下 mapper 包下，* 代表会扫描所有 xml 文件。


Mbytes
Freemarker
Restful 


