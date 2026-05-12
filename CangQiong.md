# 苍穹外卖 Java 项目完整复现教程（超详细版）

> 本教程目标：
>
> 从零开始，完整复现“苍穹外卖”Java项目。
>
> 即使是第一次接触 SpringBoot + MySQL + Redis + Vue 的开发者，也可以严格按照本文步骤一步一步完成。
>
> 本文包含：
>
> * 环境安装
> * 数据库配置
> * 后端项目搭建
> * 前端运行
> * Nginx配置
> * Redis配置
> * JWT登录
> * 文件上传
> * 微信支付（模拟）
> * Docker可选部署
> * 常见报错解决
> * 项目运行验证
>
> 全部使用 Markdown 格式。

---

# 一、项目介绍

苍穹外卖是一个典型的前后端分离外卖系统项目。

项目通常包含：

| 模块    | 技术            |
| ----- | ------------- |
| 后端    | SpringBoot    |
| 数据库   | MySQL         |
| 缓存    | Redis         |
| ORM   | MyBatis       |
| 权限    | JWT           |
| 前端管理端 | Vue           |
| 用户端   | 微信小程序         |
| 文件存储  | 阿里云OSS（可替代本地） |
| 接口文档  | Swagger       |
| 反向代理  | Nginx         |

系统角色：

* 管理员
* 商家
* 用户

主要功能：

* 登录认证
* 员工管理
* 分类管理
* 菜品管理
* 套餐管理
* 订单管理
* 地址管理
* 购物车
* 下单支付
* 数据统计

---

# 二、开发环境准备

# 1、安装 JDK

推荐版本：

* JDK 17

原因：

* SpringBoot 3.x 推荐使用 JDK17
* 更稳定
* 兼容性更好

## 下载地址

推荐使用：

* Oracle JDK
* OpenJDK

安装后验证：

```bash
java -version
```

正确输出示例：

```bash
java version "17.0.8"
```

---

# 2、安装 IntelliJ IDEA

推荐版本：

* IDEA 2023+

安装插件：

* Lombok
* MyBatisX
* Maven Helper

---

# 3、安装 Maven

推荐版本：

* Maven 3.9+

## 配置环境变量

新增：

```text
MAVEN_HOME
```

例如：

```text
D:\apache-maven-3.9.6
```

Path增加：

```text
%MAVEN_HOME%\bin
```

验证：

```bash
mvn -v
```

---

# 4、配置 Maven 镜像

打开：

```text
conf/settings.xml
```

添加阿里云镜像：

```xml
<mirrors>
    <mirror>
        <id>alimaven</id>
        <mirrorOf>*</mirrorOf>
        <name>aliyun maven</name>
        <url>https://maven.aliyun.com/repository/public</url>
    </mirror>
</mirrors>
```

---

# 5、安装 MySQL

推荐版本：

* MySQL 8.0

安装完成后：

创建数据库：

```sql
CREATE DATABASE sky_take_out DEFAULT CHARSET utf8mb4;
```

---

# 6、安装 Redis

推荐版本：

* Redis 7

启动：

```bash
redis-server
```

测试：

```bash
redis-cli
ping
```

输出：

```text
PONG
```

---

# 7、安装 Node.js

推荐版本：

* Node.js 18+

验证：

```bash
node -v
npm -v
```

---

# 8、安装 Git

验证：

```bash
git --version
```

---

# 三、创建数据库

# 1、执行SQL脚本

在项目中通常会提供：

```text
sky.sql
```

使用 Navicat 或 IDEA Database 执行。

执行后会生成：

* employee
* category
* dish
* setmeal
* orders
* shopping_cart
* address_book

等表。

---

# 2、检查数据库编码

必须：

```text
utf8mb4
```

否则可能乱码。

查看：

```sql
SHOW VARIABLES LIKE 'character%';
```

---

# 四、导入后端项目

# 1、打开 IDEA

选择：

```text
Open
```

打开项目根目录。

---

# 2、等待 Maven 下载依赖

第一次较慢。

可能需要：

```text
Reload Maven Project
```

---

# 3、检查项目结构

标准结构：

```text
sky-take-out
 ├── sky-common
 ├── sky-pojo
 ├── sky-server
```

---

# 五、配置 application.yml

位置：

```text
sky-server/src/main/resources/application.yml
```

示例：

```yml
server:
  port: 8080

spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/sky_take_out?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8
    username: root
    password: 123456

  redis:
    host: localhost
    port: 6379
    database: 0

mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.sky.entity
  configuration:
    map-underscore-to-camel-case: true
```

---

# 六、配置 JWT

JWT作用：

* 用户登录后生成 token
* 后续请求携带 token
* 服务端验证身份

配置：

```yml
sky:
  jwt:
    admin-secret-key: itcast
    admin-ttl: 7200000
    admin-token-name: token
```

---

# 七、启动 Redis

必须保证 Redis 正在运行。

否则报错：

```text
Unable to connect to Redis
```

Windows：

```bash
redis-server.exe redis.windows.conf
```

Linux：

```bash
systemctl start redis
```

---

# 八、启动项目

启动类：

```text
SkyApplication
```

运行后输出：

```text
Started SkyApplication
```

说明启动成功。

---

# 九、Swagger接口测试

访问：

```text
http://localhost:8080/doc.html
```

或者：

```text
http://localhost:8080/swagger-ui.html
```

可以测试所有接口。

---

# 十、管理员登录功能实现

# 1、登录流程

流程：

```text
前端输入账号密码
        ↓
后端查询数据库
        ↓
密码验证
        ↓
生成JWT Token
        ↓
返回前端
        ↓
前端保存Token
```

---

# 2、Controller层

```java
@RestController
@RequestMapping("/admin/employee")
public class EmployeeController {

    @Autowired
    private EmployeeService employeeService;

    @PostMapping("/login")
    public Result<EmployeeLoginVO> login(@RequestBody EmployeeLoginDTO employeeLoginDTO){
        return Result.success(employeeService.login(employeeLoginDTO));
    }
}
```

---

# 3、Service层

```java
public EmployeeLoginVO login(EmployeeLoginDTO dto){

    Employee employee = employeeMapper.getByUsername(dto.getUsername());

    if(employee == null){
        throw new AccountNotFoundException();
    }

    if(!employee.getPassword().equals(DigestUtils.md5DigestAsHex(dto.getPassword().getBytes()))){
        throw new PasswordErrorException();
    }

    String token = JwtUtil.createJWT(secretKey, ttl, claims);

    return EmployeeLoginVO.builder()
            .id(employee.getId())
            .token(token)
            .build();
}
```

---

# 十一、统一结果返回

创建 Result 类：

```java
@Data
public class Result<T> {

    private Integer code;
    private String msg;
    private T data;

    public static <T> Result<T> success(T object){
        Result<T> result = new Result<>();
        result.code = 1;
        result.data = object;
        return result;
    }

    public static <T> Result<T> error(String msg){
        Result result = new Result();
        result.code = 0;
        result.msg = msg;
        return result;
    }
}
```

---

# 十二、统一异常处理

创建全局异常处理器：

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public Result exceptionHandler(Exception ex){
        ex.printStackTrace();
        return Result.error(ex.getMessage());
    }
}
```

---

# 十三、JWT拦截器

# 1、创建拦截器

```java
@Component
public class JwtTokenAdminInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {

        String token = request.getHeader("token");

        try {
            JwtUtil.parseJWT(secretKey, token);
            return true;
        } catch (Exception ex) {
            response.setStatus(401);
            return false;
        }
    }
}
```

---

# 2、注册拦截器

```java
@Configuration
public class WebMvcConfiguration extends WebMvcConfigurationSupport {

    @Autowired
    private JwtTokenAdminInterceptor jwtTokenAdminInterceptor;

    @Override
    protected void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(jwtTokenAdminInterceptor)
                .addPathPatterns("/admin/**")
                .excludePathPatterns("/admin/employee/login");
    }
}
```

---

# 十四、员工管理模块

功能：

* 新增员工
* 删除员工
* 修改员工
* 分页查询
* 状态启用禁用

---

# 1、新增员工接口

```java
@PostMapping
public Result save(@RequestBody EmployeeDTO employeeDTO){
    employeeService.save(employeeDTO);
    return Result.success();
}
```

---

# 2、分页查询

使用 PageHelper：

```java
PageHelper.startPage(page, pageSize);
Page<Employee> pageInfo = employeeMapper.pageQuery(employeePageQueryDTO);
```

---

# 十五、分类管理

分类：

* 菜品分类
* 套餐分类

接口：

* 新增分类
* 修改分类
* 删除分类
* 分类分页

---

# 十六、文件上传功能

# 1、本地上传

```java
@PostMapping("/upload")
public Result<String> upload(MultipartFile file) throws Exception {

    String originalFilename = file.getOriginalFilename();

    String extension = originalFilename.substring(originalFilename.lastIndexOf("."));

    String objectName = UUID.randomUUID() + extension;

    file.transferTo(new File("D:/upload/" + objectName));

    return Result.success(objectName);
}
```

---

# 2、阿里云OSS上传

添加依赖：

```xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-sdk-oss</artifactId>
    <version>3.15.1</version>
</dependency>
```

配置：

```yml
aliyun:
  oss:
    endpoint: xxx
    accessKeyId: xxx
    accessKeySecret: xxx
    bucketName: xxx
```

---

# 十七、菜品管理

功能：

* 新增菜品
* 菜品分页
* 修改菜品
* 删除菜品
* 起售停售

---

# 十八、套餐管理

功能：

* 套餐新增
* 套餐分页
* 套餐停售
* 套餐删除

---

# 十九、Redis缓存

缓存分类数据：

```java
String key = "category_" + type;

Object object = redisTemplate.opsForValue().get(key);

if(object != null){
    return (List<Category>) object;
}

List<Category> list = categoryMapper.list(type);

redisTemplate.opsForValue().set(key, list);
```

---

# 二十、购物车功能

# 1、添加购物车

逻辑：

```text
判断是否已存在
    ↓
存在则数量+1
    ↓
不存在则新增
```

---

# 2、查看购物车

根据当前用户ID查询。

---

# 二十一、用户下单

流程：

```text
提交订单
    ↓
校验地址
    ↓
校验购物车
    ↓
生成订单
    ↓
插入订单明细
    ↓
清空购物车
```

---

# 二十二、微信支付（模拟）

一般教学项目采用模拟支付。

流程：

```text
创建订单
    ↓
生成支付二维码
    ↓
模拟支付成功
    ↓
修改订单状态
```

---

# 二十三、订单状态

常见状态：

| 状态 | 含义  |
| -- | --- |
| 1  | 待付款 |
| 2  | 待接单 |
| 3  | 已接单 |
| 4  | 派送中 |
| 5  | 已完成 |
| 6  | 已取消 |

---

# 二十四、定时任务

# 1、开启定时任务

```java
@EnableScheduling
```

---

# 2、自动取消超时订单

```java
@Scheduled(cron = "0 * * * * ?")
public void processTimeoutOrder(){

}
```

---

# 二十五、数据统计

统计：

* 营业额
* 用户数
* 订单数
* Top10菜品

---

# 二十六、导入前端项目

管理端通常为 Vue 项目。

进入前端目录：

```bash
cd sky-admin
```

安装依赖：

```bash
npm install
```

启动：

```bash
npm run dev
```

默认地址：

```text
http://localhost:5173
```

---

# 二十七、配置前端代理

vite.config.js：

```js
server: {
  proxy: {
    '/api': {
      target: 'http://localhost:8080',
      changeOrigin: true
    }
  }
}
```

---

# 二十八、Nginx配置

# 1、安装Nginx

启动：

```bash
nginx
```

---

# 2、配置反向代理

```nginx
server {
    listen 80;

    location /api/ {
        proxy_pass http://localhost:8080/;
    }

    location / {
        root html;
        index index.html;
    }
}
```

---

# 二十九、Linux部署

# 1、安装Java

```bash
java -version
```

---

# 2、上传Jar包

```bash
scp sky-server.jar root@服务器IP:/usr/local/
```

---

# 3、启动项目

```bash
nohup java -jar sky-server.jar > log.txt 2>&1 &
```

查看日志：

```bash
tail -f log.txt
```

---

# 三十、Docker部署（可选）

# 1、编写 Dockerfile

```dockerfile
FROM openjdk:17

COPY sky-server.jar app.jar

ENTRYPOINT ["java","-jar","/app.jar"]
```

---

# 2、构建镜像

```bash
docker build -t sky .
```

---

# 3、运行容器

```bash
docker run -d -p 8080:8080 sky
```

---

# 三十一、常见错误解决

# 1、数据库连接失败

错误：

```text
Communications link failure
```

解决：

* MySQL未启动
* 用户名密码错误
* 端口错误

---

# 2、Redis连接失败

错误：

```text
Unable to connect to Redis
```

解决：

* Redis未启动
* 端口错误

---

# 3、端口占用

错误：

```text
Port 8080 was already in use
```

解决：

Windows：

```bash
netstat -ano | findstr 8080
```

结束进程：

```bash
taskkill /PID 进程号 /F
```

---

# 4、JWT失效

错误：

```text
401 Unauthorized
```

解决：

* token过期
* token未携带
* 请求头错误

---

# 三十二、完整项目运行检查

# 后端检查

访问：

```text
http://localhost:8080/doc.html
```

能够打开Swagger。

---

# 前端检查

访问：

```text
http://localhost:5173
```

能够进入登录页面。

---

# 登录测试

输入：

```text
admin
123456
```

能够成功登录。

---

# 数据库检查

查看：

```sql
SELECT * FROM employee;
```

确认数据正常。

---

# Redis检查

```bash
redis-cli
keys *
```

确认缓存已生成。

---

# 三十三、推荐学习顺序

建议顺序：

```text
SpringBoot
    ↓
MySQL
    ↓
MyBatis
    ↓
Redis
    ↓
JWT
    ↓
Vue
    ↓
Nginx
    ↓
Docker
```

---

# 三十四、项目最终目录结构

```text
sky-take-out
│
├── sky-common
│
├── sky-pojo
│
├── sky-server
│   ├── controller
│   ├── service
│   ├── mapper
│   ├── entity
│   ├── dto
│   ├── vo
│   ├── config
│   ├── interceptor
│   └── utils
│
├── sky-admin
│
└── sql
```

---

# 三十五、推荐优化方向

后续可以继续扩展：

* Docker Compose
* Kubernetes
* Elasticsearch
* RabbitMQ
* XXL-Job
* SpringCloud
* 微服务拆分
* AI推荐系统
* OCR菜单识别

---

# 三十六、完整复现建议

严格按照以下顺序：

```text
1. 安装JDK
2. 安装IDEA
3. 安装MySQL
4. 安装Redis
5. 安装Node.js
6. 导入数据库
7. 配置application.yml
8. 启动Redis
9. 启动后端
10. 启动前端
11. 测试登录
12. 测试CRUD
13. 测试缓存
14. 测试订单
15. 部署Linux
```

---

# 三十七、最终效果

完成后你将获得：

* 完整SpringBoot项目开发经验
* 前后端分离项目经验
* Redis缓存经验
* JWT认证经验
* MyBatis开发经验
* Linux部署经验
* Docker部署经验

已经具备中小型Java项目开发能力。

---

# 三十八、附录：推荐依赖版本

```xml
<properties>
    <java.version>17</java.version>
    <spring-boot.version>3.2.5</spring-boot.version>
    <mysql.version>8.0.33</mysql.version>
    <mybatis.version>3.5.13</mybatis.version>
    <redis.version>3.2.5</redis.version>
</properties>
```

---

# 三十九、附录：推荐学习资料

推荐重点学习：

* SpringBoot自动配置
* MyBatis动态SQL
* Redis缓存设计
* JWT原理
* Vue组件通信
* Linux命令
* Docker基础

---

# 四十、结束语

至此，苍穹外卖项目已经能够完整复现。

如果严格按照本文操作：

* 环境不会缺失
* 配置不会遗漏
* 数据库不会错误
* 前后端能够正常联调
* 可以成功部署运行

建议：

第一次完整跑通项目。

第二次开始尝试：

* 自己独立写模块
* 增加新功能
* 优化代码结构
* 做性能优化

这样才能真正掌握 Java 企业级开发。
