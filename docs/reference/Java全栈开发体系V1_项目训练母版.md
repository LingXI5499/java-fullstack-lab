# Java 全栈开发体系 V1 —— 项目训练母版

> 本文件是 java-fullstack-lab 的**需求来源（只读）**，不是流水线本体。
> 它与本仓库规范的对齐关系与三处有意偏离，见同目录 [README.md](README.md)。

> **定位**：这不是“技术名词清单”，而是一套可以反复用于真实项目开发、训练、讲解、答辩和后续升级的 **Java 全栈基础工程体系**。  
> 当前阶段的目标不是继续堆 Redis、MQ、微服务等高级技术，而是把现有技术真正串成完整开发链路，并通过大量项目训练形成稳定的工程能力。

---

# 1. 这套体系到底是什么

当前技术体系可以概括为：

> **JDK 17 + Maven + Spring Boot + Spring MVC + MyBatis + MySQL + JWT + Vue + Element Plus + ECharts + JUnit + Apifox/Postman + OSS + Linux/Nginx**

它能够支撑一套完整的：

- 前后端分离 Web 项目
- RESTful API
- 用户登录认证
- CRUD 业务开发
- 条件查询
- 分页查询
- 多表查询
- 文件上传
- 数据统计
- 异常处理
- 日志管理
- 事务管理
- 单元测试
- Maven 多模块
- 前后端打包
- Linux 部署

因此，这套体系已经足够作为：

> **Java 后端 / Java 全栈开发的 V1 基础工程底座。**

现阶段最重要的不是继续增加技术数量，而是训练四件事：

1. **会讲**
2. **会设计**
3. **会写**
4. **会调试和部署**

---

# 2. 整体技术架构

完整请求链路可以抽象成：

```text
用户
 ↓
Browser
 ↓
Vue
 ↓
Axios
 ↓
HTTP / JSON
 ↓
RESTful API
 ↓
Spring Boot
 ↓
Spring MVC
 ↓
Controller
 ↓
Service
 ↓
Mapper
 ↓
MyBatis
 ↓
MySQL
```

这是整个体系最重要的“主干”。

以后无论开发：

- 学生管理
- 药店管理
- 图书管理
- 博客
- 招聘系统
- 课程系统
- 订单系统

本质上都在不断重复这条链路。

---

# 3. 技术体系分层

---

## 3.1 客户端层

用户通过浏览器访问系统。

```text
User
 ↓
Browser
```

常见客户端：

- PC 管理后台
- 普通 Web 页面
- H5
- 移动端浏览器

这一层主要负责：

> **用户操作与信息展示。**

---

# 4. 前端开发体系

当前前端技术：

```text
Vue
├── Vue Component
├── Vue Router
├── Axios
├── Element Plus
└── ECharts
```

---

## 4.1 Vue

Vue 是当前前端项目的核心框架。

需要掌握：

- 组件 Component
- `template`
- `script`
- `style`
- `ref`
- `reactive`
- `computed`
- `watch`
- `v-if`
- `v-for`
- `v-model`
- `props`
- `emit`
- 生命周期

目标不是成为专业前端工程师，而是：

> 能独立完成 Java 项目所需要的后台管理前端。

---

## 4.2 Element Plus

负责快速构建后台 UI。

常用组件：

- Button
- Input
- Select
- Form
- Table
- Pagination
- Dialog
- Menu
- Upload
- Message
- DatePicker

典型页面：

```text
用户管理页面
├── 搜索区域
├── 新增按钮
├── 用户表格
├── 分页
└── 新增/编辑弹窗
```

---

## 4.3 Vue Router

负责页面路由。

例如：

```text
/login
/dashboard
/users
/products
/orders
/profile
```

主要掌握：

- 路由定义
- 页面跳转
- 动态路由参数
- 路由守卫
- 登录状态判断

---

## 4.4 Axios

负责前后端 HTTP 通信。

例如：

```text
Vue
 ↓
Axios
 ↓
GET /api/users
 ↓
Spring Boot
```

建议统一封装：

```text
src/utils/request.js
```

处理：

- baseURL
- Token
- 请求拦截
- 响应拦截
- 错误处理

---

## 4.5 ECharts

主要用于：

- 柱状图
- 折线图
- 饼图
- 仪表盘
- 数据统计页面

例如：

```text
Dashboard
├── 用户总数
├── 今日新增
├── 商品数量
├── 订单数量
├── 月度趋势
└── 分类占比
```

---

# 5. HTTP 与 RESTful API

前端和后端之间通过：

```text
HTTP + JSON
```

进行通信。

---

## 5.1 RESTful

推荐统一使用：

| 功能 | HTTP Method | URL |
|---|---|---|
| 新增用户 | POST | `/users` |
| 查询用户 | GET | `/users/{id}` |
| 修改用户 | PUT | `/users/{id}` |
| 删除用户 | DELETE | `/users/{id}` |
| 用户列表 | GET | `/users` |
| 分页查询 | GET | `/users?page=1&pageSize=10` |

避免：

```text
/addUser
/deleteUser
/updateUser
```

RESTful 的核心思想是：

> **URL 表示资源，HTTP Method 表示操作。**

---

# 6. Spring Boot 后端体系

Spring Boot 是整个后端运行核心。

```text
Spring Boot
├── Spring MVC
├── IOC
├── DI
├── Bean
├── 自动配置
├── 起步依赖
├── 配置文件
└── Web Server
```

---

# 7. Spring Boot 配置体系

主要配置文件：

```text
application.yml
application.yaml
application.properties
```

推荐项目统一使用：

```text
application.yml
```

多环境可以进一步拆分：

```text
application.yml
application-dev.yml
application-prod.yml
```

配置内容包括：

- 服务端口
- MySQL
- MyBatis
- OSS
- JWT
- 日志
- 文件上传限制

---

# 8. Spring IOC / DI / Bean

这是 Spring 的核心思想。

一句话：

> **对象不再由程序员到处 new，而是交给 Spring 容器统一创建和管理。**

例如：

```text
UserController
      ↓
UserService
      ↓
UserMapper
```

对象由 Spring 创建：

```text
Spring IOC Container
├── UserController Bean
├── UserServiceImpl Bean
└── UserMapper Proxy
```

然后通过 DI 建立依赖关系。

需要掌握：

- `@Component`
- `@Service`
- `@Controller`
- `@RestController`
- `@Autowired`
- 构造器注入
- `@Bean`
- `@Configuration`

---

# 9. Spring MVC 请求链路

核心流程：

```text
Browser
 ↓
HTTP Request
 ↓
Tomcat
 ↓
DispatcherServlet
 ↓
HandlerMapping
 ↓
HandlerAdapter
 ↓
参数解析
 ↓
Controller
 ↓
Service
 ↓
Mapper
```

常见参数：

```java
@RequestBody
@PathVariable
@RequestParam
```

例如：

```java
@PostMapping("/users")
public Result<Void> add(@RequestBody UserDTO userDTO)
```

---

# 10. Controller

Controller 是 Web 接口层。

职责：

> **接收请求 → 参数接收 → 调用 Service → 返回数据**

不应该在 Controller 中：

- 编写大量 SQL
- 编写复杂业务逻辑
- 直接操作数据库
- 写大量重复异常处理

典型：

```text
UserController
LoginController
ProductController
OrderController
FileController
StatsController
```

---

# 11. Service

Service 是业务逻辑核心。

职责：

- 业务规则
- 数据校验
- 多表业务操作
- 事务控制
- 调用 Mapper
- 组合数据

例如：

```text
创建订单
 ↓
检查商品
 ↓
检查库存
 ↓
创建订单
 ↓
创建订单明细
 ↓
更新库存
```

这些逻辑应该放在 Service。

---

# 12. Mapper

Mapper 是数据访问层。

职责：

> 定义数据库访问方法。

例如：

```java
User selectById(Long id);

List<User> selectPage(UserQuery query);

int insert(User user);

int update(User user);

int deleteById(Long id);
```

---

# 13. MyBatis

MyBatis 负责：

```text
Java Object
   ↕
SQL
   ↕
MySQL
```

核心结构：

```text
Mapper Interface
      ↓
Mapper XML
      ↓
SQL
```

需要掌握：

- `#{}` 参数绑定
- `${}` 字符串拼接
- `resultType`
- `resultMap`
- `<if>`
- `<where>`
- `<foreach>`
- 动态 SQL
- 多表映射

---

# 14. MySQL 数据库体系

当前阶段建议掌握完整基础数据库能力。

---

## 14.1 DDL

```text
CREATE
ALTER
DROP
```

---

## 14.2 DML

```text
INSERT
UPDATE
DELETE
```

---

## 14.3 DQL

```text
SELECT
```

包括：

- WHERE
- LIKE
- BETWEEN
- IN
- ORDER BY
- GROUP BY
- HAVING
- LIMIT

---

## 14.4 聚合函数

```text
COUNT
SUM
AVG
MAX
MIN
```

---

## 14.5 多表查询

重点掌握：

```text
INNER JOIN
LEFT JOIN
```

同时了解：

- 子查询
- 一对一
- 一对多
- 多对多

---

## 14.6 基础索引

当前阶段不需要深入数据库内核，但必须知道：

- PRIMARY KEY
- UNIQUE
- INDEX
- 索引为什么能提高查询速度
- 什么字段适合建立索引
- 索引会增加写入成本

---

# 15. 数据库连接池

Spring Boot 不应该每次访问数据库都重新建立连接。

而是通过：

```text
DataSource
 ↓
Connection Pool
 ↓
MySQL
```

管理数据库连接。

需要理解：

> 数据库连接是一种昂贵资源，所以应该复用连接。

---

# 16. DTO / Entity / VO / Query

这是必须正式加入当前体系的内容。

---

## 16.1 Entity

对应数据库表。

例如：

```text
tb_user
 ↓
User
```

---

## 16.2 DTO

DTO：

> Data Transfer Object

用于接收前端请求。

例如：

```java
LoginDTO
UserDTO
ProductDTO
OrderDTO
```

---

## 16.3 VO

VO：

> View Object

用于返回前端展示。

例如：

```java
UserVO
LoginVO
DashboardVO
OrderVO
```

---

## 16.4 Query / PageQuery

用于封装：

- 页码
- 每页数量
- 搜索词
- 状态
- 时间范围

例如：

```java
UserQuery
ProductQuery
PageQuery
```

---

# 17. Lombok

用于减少 Java Bean 模板代码。

常见：

```java
@Data
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
```

用途：

> 减少 getter、setter、构造器等重复代码。

---

# 18. 参数校验

建议统一加入：

```java
@Valid
@NotNull
@NotBlank
@Size
@Min
@Max
```

流程：

```text
HTTP Request
 ↓
DTO
 ↓
@Valid
 ↓
参数是否合法
```

避免 Service 到处写：

```java
if (name == null)
```

---

# 19. 统一返回结果

建议定义：

```java
Result<T>
```

例如：

```json
{
  "code": 200,
  "message": "success",
  "data": {}
}
```

所有接口统一返回：

```text
Result<T>
```

而不是每个 Controller 使用不同格式。

---

# 20. 全局异常处理

推荐：

```text
业务异常
 ↓
BusinessException
 ↓
GlobalExceptionHandler
 ↓
Result.error()
```

核心技术：

```java
@ControllerAdvice
@ExceptionHandler
```

或者：

```java
@RestControllerAdvice
```

这样 Controller 不需要大量 try/catch。

---

# 21. 日志 SLF4J

日志应该用于：

- 记录系统运行状态
- 记录业务操作
- 记录错误
- 调试问题
- 线上排错

例如：

```java
log.info("用户登录成功 userId={}", userId);

log.error("订单创建失败 orderId={}", orderId, e);
```

日志级别：

```text
TRACE
DEBUG
INFO
WARN
ERROR
```

---

# 22. 事务管理

业务操作经常需要保证：

> 要么全部成功，要么全部失败。

例如：

```text
创建订单
├── 插入订单
├── 插入订单明细
└── 扣库存
```

如果最后一步失败：

```text
前面操作全部 rollback
```

典型：

```java
@Transactional
```

---

# 23. AOP

AOP：

> Aspect-Oriented Programming

适合处理横切关注点：

- 操作日志
- 性能统计
- 审计
- 权限辅助
- 统一行为增强

典型结构：

```text
业务方法
 ↑
AOP 切面
```

例如：

```text
Controller Method
 ↓
记录开始时间
 ↓
执行方法
 ↓
记录结束时间
 ↓
统计耗时
```

---

# 24. 登录认证体系

需要理解三个基本概念：

```text
Cookie
Session
JWT
```

---

## 24.1 Cookie

Cookie 存储在浏览器。

浏览器可以自动携带 Cookie。

---

## 24.2 Session

Session 一般保存在服务端。

流程：

```text
登录
 ↓
创建 Session
 ↓
返回 Session ID
 ↓
Cookie 保存
 ↓
后续请求携带 Session ID
```

---

## 24.3 JWT

JWT 是当前前后端分离项目常用方案。

登录：

```text
username/password
 ↓
Controller
 ↓
Service
 ↓
查询数据库
 ↓
验证密码
 ↓
生成 JWT
 ↓
返回 Token
```

后续访问：

```text
Vue
 ↓
Authorization: Bearer token
 ↓
Filter / Interceptor
 ↓
解析 JWT
 ↓
校验用户
 ↓
Controller
```

---

# 25. Filter 与 Interceptor

---

## Filter

属于 Servlet 层。

常见用途：

- 编码
- 请求过滤
- Token 预处理
- 日志

---

## Interceptor

属于 Spring MVC 层。

常见用途：

- 登录认证
- 权限检查
- Controller 前后处理

需要能讲清：

```text
Filter
 ↓
DispatcherServlet
 ↓
Interceptor
 ↓
Controller
```

---

# 26. 文件上传与阿里云 OSS

典型流程：

```text
Vue
 ↓
Upload
 ↓
Spring Boot
 ↓
MultipartFile
 ↓
OSS SDK
 ↓
阿里云 OSS
 ↓
返回文件 URL
```

常见业务：

- 用户头像
- 商品图片
- 简历附件
- 文档附件
- 项目截图

---

# 27. JUnit 单元测试

JUnit 不应该只是“学过一个注解”。

需要真正用于测试：

- Service
- Utils
- 数据转换
- 业务规则

常见：

```java
@Test
```

项目中：

```text
src/test/java/
```

建议后续每个核心 Service 至少有基本测试。

---

# 28. Apifox / Postman

作用：

> 在前端还没开发完成之前，就可以独立测试后端 API。

流程：

```text
Controller 开发完成
 ↓
启动 Spring Boot
 ↓
Apifox / Postman
 ↓
发送 HTTP 请求
 ↓
查看 Response
```

需要训练：

- GET
- POST
- PUT
- DELETE
- Header
- JSON Body
- Query
- PathVariable
- JWT Token
- 文件上传

---

# 29. Maven

Maven 是 Java 项目的构建和依赖管理工具。

需要掌握：

```text
dependency
plugin
repository
lifecycle
module
parent
```

常见命令：

```bash
mvn clean
mvn compile
mvn test
mvn package
mvn install
```

---

# 30. Maven 多模块

多模块的核心不是“把目录拆多”。

而是：

> 按职责拆分工程，并建立模块依赖关系。

例如：

```text
project
├── project-common
├── project-pojo
└── project-server
```

---

# 31. Maven 继承

解决：

> 公共 Maven 配置重复。

例如：

```text
parent pom
 ↓
child module
```

父项目可以统一：

- Java 版本
- Spring Boot 版本
- 依赖版本
- 插件版本

---

# 32. Maven 聚合

解决：

> 一次构建多个模块。

例如：

```xml
<modules>
    <module>project-common</module>
    <module>project-pojo</module>
    <module>project-server</module>
</modules>
```

然后：

```bash
mvn clean package
```

即可构建整个项目。

---

# 33. Maven 私服

主要理解用途即可。

典型：

```text
Developer
 ↓
Maven
 ↓
Company Nexus Repository
 ↓
Maven Central
```

解决：

- 企业内部 jar
- 私有组件
- 依赖缓存
- 统一依赖管理
- 下载加速

---

# 34. Git

建议正式加入基础工程体系。

至少掌握：

```bash
git init
git add
git commit
git push
git pull
git branch
git merge
```

理解：

- repository
- commit
- branch
- remote
- merge
- conflict

---

# 35. Linux

部署阶段必须掌握基础 Linux。

例如：

```bash
ls
cd
pwd
cp
mv
rm
mkdir
cat
tail
grep
ps
kill
chmod
```

以及：

```bash
java -jar app.jar
```

---

# 36. Nginx

基础阶段主要用于：

```text
Browser
 ↓
Nginx
 ├── Vue dist
 └── /api → Spring Boot
```

掌握：

- 静态资源部署
- 反向代理
- 基础 location
- 前后端统一入口

---

# 37. 前端打包

Vue：

```bash
npm run build
```

生成：

```text
dist/
```

然后部署到：

```text
Nginx
```

---

# 38. 后端打包

Spring Boot：

```bash
mvn clean package
```

生成：

```text
target/project.jar
```

Linux：

```bash
java -jar project.jar
```

---

# 39. 标准项目目录

推荐以后绝大多数训练项目统一采用类似结构。

```text
project-root/
│
├── docs/
│   ├── requirement.md
│   ├── database.md
│   ├── api.md
│   └── deployment.md
│
├── sql/
│   ├── schema.sql
│   ├── data.sql
│   └── test-data.sql
│
├── apifox/
│
├── frontend/
│   │
│   ├── src/
│   │   ├── api/
│   │   │   ├── user.js
│   │   │   ├── login.js
│   │   │   ├── product.js
│   │   │   ├── order.js
│   │   │   ├── stats.js
│   │   │   └── file.js
│   │   │
│   │   ├── assets/
│   │   │
│   │   ├── components/
│   │   │   ├── Pagination.vue
│   │   │   ├── UploadFile.vue
│   │   │   └── ChartCard.vue
│   │   │
│   │   ├── layout/
│   │   │   ├── Header.vue
│   │   │   ├── Sidebar.vue
│   │   │   └── MainLayout.vue
│   │   │
│   │   ├── router/
│   │   │   └── index.js
│   │   │
│   │   ├── stores/
│   │   │
│   │   ├── utils/
│   │   │   ├── request.js
│   │   │   ├── token.js
│   │   │   └── format.js
│   │   │
│   │   ├── views/
│   │   │   ├── login/
│   │   │   │   └── Login.vue
│   │   │   │
│   │   │   ├── dashboard/
│   │   │   │   └── Dashboard.vue
│   │   │   │
│   │   │   ├── user/
│   │   │   │   ├── UserList.vue
│   │   │   │   └── UserForm.vue
│   │   │   │
│   │   │   ├── product/
│   │   │   ├── order/
│   │   │   ├── upload/
│   │   │   └── profile/
│   │   │
│   │   ├── App.vue
│   │   └── main.js
│   │
│   ├── package.json
│   └── dist/
│
└── backend/
    │
    ├── src/main/java/com/example/project/
    │   │
    │   ├── ProjectApplication.java
    │   │
    │   ├── controller/
    │   │   ├── LoginController.java
    │   │   ├── UserController.java
    │   │   ├── ProductController.java
    │   │   ├── OrderController.java
    │   │   ├── FileController.java
    │   │   └── StatsController.java
    │   │
    │   ├── service/
    │   │   ├── UserService.java
    │   │   ├── ProductService.java
    │   │   └── OrderService.java
    │   │
    │   ├── service/impl/
    │   │   ├── UserServiceImpl.java
    │   │   ├── ProductServiceImpl.java
    │   │   └── OrderServiceImpl.java
    │   │
    │   ├── mapper/
    │   │   ├── UserMapper.java
    │   │   ├── ProductMapper.java
    │   │   └── OrderMapper.java
    │   │
    │   ├── entity/
    │   │   ├── User.java
    │   │   ├── Product.java
    │   │   └── Order.java
    │   │
    │   ├── dto/
    │   │   ├── LoginDTO.java
    │   │   ├── UserDTO.java
    │   │   ├── ProductDTO.java
    │   │   └── OrderDTO.java
    │   │
    │   ├── vo/
    │   │   ├── LoginVO.java
    │   │   ├── UserVO.java
    │   │   ├── OrderVO.java
    │   │   └── DashboardVO.java
    │   │
    │   ├── query/
    │   │   ├── PageQuery.java
    │   │   ├── UserQuery.java
    │   │   └── OrderQuery.java
    │   │
    │   ├── common/
    │   │   ├── result/
    │   │   │   └── Result.java
    │   │   ├── constants/
    │   │   └── enums/
    │   │
    │   ├── config/
    │   │   ├── WebMvcConfig.java
    │   │   ├── MybatisConfig.java
    │   │   ├── CorsConfig.java
    │   │   └── OssConfig.java
    │   │
    │   ├── interceptor/
    │   │   └── JwtInterceptor.java
    │   │
    │   ├── filter/
    │   │   └── TokenFilter.java
    │   │
    │   ├── exception/
    │   │   ├── BusinessException.java
    │   │   └── GlobalExceptionHandler.java
    │   │
    │   ├── aspect/
    │   │   └── OperationLogAspect.java
    │   │
    │   ├── utils/
    │   │   ├── JwtUtils.java
    │   │   ├── OssUtils.java
    │   │   ├── PasswordUtils.java
    │   │   └── DateUtils.java
    │   │
    │   └── annotation/
    │
    ├── src/main/resources/
    │   ├── application.yml
    │   ├── application-dev.yml
    │   ├── application-prod.yml
    │   │
    │   └── mapper/
    │       ├── UserMapper.xml
    │       ├── ProductMapper.xml
    │       └── OrderMapper.xml
    │
    ├── src/test/java/
    │   ├── UserServiceTest.java
    │   └── OrderServiceTest.java
    │
    ├── pom.xml
    └── target/
```

---

# 40. 每个目录的职责

| 目录 | 主要职责 |
|---|---|
| controller | HTTP 接口 |
| service | 业务接口 |
| service.impl | 业务实现 |
| mapper | 数据访问接口 |
| mapper XML | SQL |
| entity | 数据库实体 |
| dto | 请求数据 |
| vo | 响应数据 |
| query | 查询条件 |
| common | 公共代码 |
| config | Spring 配置 |
| interceptor | MVC 拦截 |
| filter | Servlet 请求过滤 |
| exception | 异常 |
| aspect | AOP |
| utils | 工具类 |
| annotation | 自定义注解 |
| resources | 配置和 XML |
| test | 测试代码 |

---

# 41. 一个标准业务模块应该是什么样

以用户管理为例：

```text
User
│
├── UserController
├── UserService
├── UserServiceImpl
├── UserMapper
├── UserMapper.xml
├── User
├── UserDTO
├── UserVO
└── UserQuery
```

---

# 42. 用户新增完整调用链

```text
UserForm.vue
 ↓
user.js
 ↓
Axios
 ↓
POST /users
 ↓
UserController
 ↓
UserDTO
 ↓
UserService
 ↓
UserServiceImpl
 ↓
User Entity
 ↓
UserMapper
 ↓
UserMapper.xml
 ↓
INSERT INTO user...
 ↓
MySQL
```

这条链必须训练到能够脱口而出。

---

# 43. 分页查询完整链路

```text
Vue
 ↓
page / pageSize / keyword / status
 ↓
Axios
 ↓
GET /users
 ↓
UserQuery
 ↓
Controller
 ↓
Service
 ↓
Mapper
 ↓
MyBatis XML
 ↓
SELECT ...
WHERE ...
ORDER BY ...
LIMIT ...
 ↓
MySQL
 ↓
List<User>
 ↓
VO
 ↓
Result<PageResult<UserVO>>
 ↓
Vue Table
```

---

# 44. 登录完整链路

```text
Login.vue
 ↓
username + password
 ↓
POST /login
 ↓
LoginDTO
 ↓
LoginController
 ↓
UserService
 ↓
UserMapper
 ↓
MySQL
 ↓
校验用户
 ↓
生成 JWT
 ↓
LoginVO
 ↓
Token
 ↓
Vue 保存 Token
 ↓
Axios Request Interceptor
 ↓
Authorization Header
 ↓
JwtInterceptor
 ↓
验证通过
 ↓
Controller
```

---

# 45. 文件上传完整链路

```text
Element Plus Upload
 ↓
MultipartFile
 ↓
FileController
 ↓
OssUtils
 ↓
Alibaba Cloud OSS
 ↓
URL
 ↓
Result<String>
 ↓
Vue
```

---

# 46. 数据统计完整链路

```text
Dashboard.vue
 ↓
Axios
 ↓
GET /stats/dashboard
 ↓
StatsController
 ↓
StatsService
 ↓
Mapper
 ↓
COUNT / SUM / GROUP BY
 ↓
MySQL
 ↓
DashboardVO
 ↓
ECharts
```

---

# 47. 一个项目应该按照什么顺序开发

以后可以把所有项目固定为：

```text
1. 需求分析
 ↓
2. 用例分析
 ↓
3. 业务流程
 ↓
4. 数据库设计
 ↓
5. API 设计
 ↓
6. 创建 Maven / Vue 工程
 ↓
7. Entity / DTO / VO
 ↓
8. Mapper
 ↓
9. Service
 ↓
10. Controller
 ↓
11. 异常处理
 ↓
12. 日志
 ↓
13. 事务
 ↓
14. 登录认证
 ↓
15. 文件上传
 ↓
16. JUnit
 ↓
17. Apifox
 ↓
18. Vue 页面
 ↓
19. 前后端联调
 ↓
20. ECharts
 ↓
21. 打包
 ↓
22. Linux 部署
 ↓
23. 测试验收
```

---

# 48. 当前阶段能力等级

建议以后不再说：

> “我学过 Spring Boot。”

而改成能力等级。

| 等级 | 标准 |
|---|---|
| L0 | 听过 |
| L1 | 能解释是什么 |
| L2 | 能看懂代码 |
| L3 | 能独立开发 |
| L4 | 能设计、Debug、优化 |
| L5 | 能给别人完整讲清楚 |

当前阶段目标：

> **整个 Java Full Stack V1 达到 L3，核心链路达到 L4。**

核心链路：

```text
Vue
 ↓
HTTP
 ↓
Spring MVC
 ↓
Controller
 ↓
Service
 ↓
Mapper
 ↓
MyBatis
 ↓
MySQL
```

---

# 49. 当前暂时不需要加入的技术

当前阶段不建议急着加入：

- Redis
- RabbitMQ
- Kafka
- Elasticsearch
- Spring Cloud
- 微服务
- 分布式事务
- Docker / Kubernetes 深度体系
- 分库分表

原因不是这些技术不重要。

而是：

> **先把单体 Web 工程真正练熟，再升级架构。**

否则容易形成：

```text
知道很多技术名称
≠
真正会开发
```

---

# 50. 这套技术体系可以做什么项目

下面这些项目都可以只用当前技术体系完成。

---

## 第一梯度：CRUD 基础训练

适合训练：

- Controller
- Service
- Mapper
- MyBatis
- MySQL
- Vue
- RESTful

### 1. 学生信息管理系统

功能：

- 学生 CRUD
- 班级
- 专业
- 成绩
- 条件搜索
- 分页

---

### 2. 图书管理系统

功能：

- 图书管理
- 分类管理
- 借阅
- 归还
- 用户
- 借阅记录

---

### 3. 通讯录管理系统

功能：

- 联系人
- 分类
- 搜索
- 分页
- 备注

---

### 4. 商品管理系统

功能：

- 商品
- 分类
- 库存
- 图片
- 状态

---

### 5. 员工管理系统

功能：

- 员工
- 部门
- 职位
- 入职
- 状态

---

# 51. 第二梯度：业务逻辑训练

重点加入：

- DTO
- VO
- 多表查询
- 事务
- 异常
- JWT

### 6. 药店管理系统

模块：

- 药品
- 药品分类
- 供应商
- 会员
- 销售
- 库存
- 订单
- 统计

---

### 7. 宿舍管理系统

模块：

- 学生
- 宿舍楼
- 房间
- 入住
- 调宿
- 报修
- 访客

---

### 8. 课程管理系统

模块：

- 学生
- 教师
- 课程
- 选课
- 成绩
- 课表

---

### 9. 仓库管理系统

模块：

- 商品
- 仓库
- 入库
- 出库
- 库存
- 供应商

---

### 10. 健身训练管理系统

模块：

- 用户
- 训练计划
- 训练动作
- 训练记录
- 体重记录
- ECharts 趋势

---

# 52. 第三梯度：综合 Web 项目

重点训练：

- 登录认证
- 文件上传
- ECharts
- 多模块
- 部署

### 11. 博客系统

模块：

- 用户
- 文章
- 分类
- 标签
- 评论
- 图片
- 搜索
- 后台管理

---

### 12. 个人知识库

模块：

- 文档
- 分类
- 标签
- Markdown
- 搜索
- 文件附件
- 管理后台

---

### 13. 新闻资讯管理系统

模块：

- 新闻
- 分类
- 来源
- 标签
- 图片
- 后台审核
- 统计

---

### 14. 校园二手交易平台

模块：

- 用户
- 商品
- 分类
- 商品图片
- 发布
- 下架
- 订单
- 个人中心

---

### 15. 招聘管理系统

模块：

- 企业
- 岗位
- 简历
- 投递
- 面试
- 状态
- 文件上传
- 数据统计

---

# 53. 第四梯度：接近企业后台项目

### 16. OA 办公管理系统

模块：

- 用户
- 部门
- 员工
- 请假
- 审批
- 公告
- 文件

---

### 17. 订单管理系统

模块：

- 用户
- 商品
- 订单
- 订单项
- 状态
- 统计
- 事务

---

### 18. 电商后台管理系统

模块：

- 用户
- 商品
- 分类
- SKU 简化
- 订单
- 图片
- 统计
- 权限基础

---

### 19. CRM 客户管理系统

模块：

- 客户
- 联系人
- 销售人员
- 跟进记录
- 商机
- 状态统计

---

### 20. 项目任务管理系统

模块：

- 用户
- 项目
- 成员
- 任务
- 状态
- 截止时间
- 进度统计

---

# 54. 更适合结合 AI Coding 的项目

这类项目的价值在于：

> 业务由你设计，AI 帮你提高编码效率。

### 21. AI 学习任务管理平台

当前技术负责：

- 用户
- 课程
- 知识点
- 学习记录
- 任务
- 数据统计

以后再加入 AI。

---

### 22. AI 知识工作台基础版

当前阶段先实现：

- 用户
- 文档
- 分类
- 文件
- 项目
- 任务
- 搜索
- 工作台

未来增加：

- RAG
- Agent
- Embedding

---

### 23. AI 简历投递管理系统

当前实现：

- 公司
- 岗位
- JD
- 简历
- 投递记录
- 状态
- 面试
- 截图
- 附件
- ECharts

以后加入：

- AI JD 分析
- AI 简历匹配
- AI 面试题

---

### 24. 个人技术知识平台

例如：

> 星雨笔录

当前体系已经可以完成：

- 教程
- 博客
- 项目
- 标签
- 搜索
- 用户
- 后台
- 文件
- 数据统计
- 登录认证

以后再升级 AI。

---

# 55. 推荐训练顺序

如果要批量做项目，不建议随机选。

推荐：

```text
项目 1
学生管理系统
 ↓
项目 2
图书管理系统
 ↓
项目 3
药店管理系统
 ↓
项目 4
仓库管理系统
 ↓
项目 5
课程管理系统
 ↓
项目 6
博客系统
 ↓
项目 7
招聘管理系统
 ↓
项目 8
订单系统
 ↓
项目 9
项目任务管理系统
 ↓
项目 10
个人知识平台
```

其中：

```text
1~2
练 CRUD

3~5
练业务建模

6~7
练完整前后端

8
练事务与多表

9
练复杂业务

10
形成个人完整作品
```

---

# 56. 最终能力目标

当一个陌生需求出现，例如：

> “开发一个课程管理系统。”

你应该能够直接在脑中形成：

```text
需求
 ↓
业务模块
 ↓
ER
 ↓
数据库
 ↓
API
 ↓
Controller
 ↓
Service
 ↓
Mapper
 ↓
Vue
 ↓
JWT
 ↓
测试
 ↓
部署
```

最终达到：

> **需求出来 → 能设计 → 能编码 → 能测试 → 能部署 → 能解释。**

这才是当前这套 Java 全栈开发体系真正的目标。

---

# 57. 一句话总结

当前阶段不要继续疯狂扩展技术栈。

应该围绕：

```text
JDK 17
+
Maven
+
Spring Boot
+
Spring MVC
+
MyBatis
+
MySQL
+
JWT
+
Vue
+
Element Plus
+
ECharts
+
JUnit
+
Apifox
+
OSS
+
Git
+
Linux
+
Nginx
```

反复完成真实项目。

当这些技术真正成为你的“默认开发语言”以后，再向：

```text
Redis
MQ
Spring Security
Docker
微服务
分布式
Agent
AI 应用架构
```

升级。

这套 **Java Full Stack V1** 就是后续所有进阶技术的地基。
