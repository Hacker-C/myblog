---
title: Vue + SpringBoot 前后端分离开发手册
date: 2021-12-5 15:20:00
tags:
  - Vue
  - SpringBoot
layout: post
subtitle: "个人记录的一个前后端相关配置文档，开发时方便查阅。"
author: "MurphyChen"
---

## 1. SpringBoot

Spring Boot 是基于 Spring 的一套全新的开源框架，去除了大量的 XML 配置文件，大大简化 Spring 应用的搭建和开发过程，简化了复杂的依赖管理。

### 1.1 原则

约定大于配置，依赖自动配置好（pom.xml），避免配置地狱。  

### 1.2 各层业务逻辑

```
Controller-->service接口-->serviceImpl-->dao接口-->daoImpl-->mapper-->db
```

在具体的项目中，其流程为：Controller层调用Service层的方法，Service层调用Dao层中的方法，其中调用的参数是使用Entity层进行传递的。总的来说这样每层做什么的分类只是为了使业务逻辑更加清晰，写代码更加方便，所以有时候也需要根据具体情况来，但是大体的都是这样处理的，因为它其实就是提供一种规则，让你把相同类型的代码放在一起，这样就形成了层次，从而达到分层解耦、复用、便于测试和维护的目的。

### 1.3 接口开发简单步骤

1. 建立实体类，跟数据库表字段保持一致；
2. 建立 mapper 接口，写具体的 sql 语句，以及这个 sql 语句要定义要操作数据库的动作；
4. 建立 service 类，处理业务逻辑；
5. 在 controller 类中创建接口，展示处理的结果。

### 1.4 application.yml 配置文件

```yml
# 应用名称
spring:
  application:
    name: springboot-admin
  datasource:
    # 数据库驱动
    driver-class-name: com.mysql.cj.jdbc.Driver
    # 数据库连接地址
    url: jdbc:mysql://127.0.0.1:3306/vue-admin?serverTimezone=UTC&useUnicode=true&characterEncoding=utf8
    # 数据库登录名
    username: root
    # 数据库密码
    password: '0316'
# 应用服务 WEB 访问端口
server:
  port: 8081
# 指定Mybatis的实体目录
mybatis:
  type-aliases-package: com.example.springbootadmin.entity
# mybatis 控制台打印日志
mybatis:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

### 1.5 各层文件配置 demo

Spring boot 启动类配置：
```java
@SpringBootApplication
// 把Mapper里的东西注入到Spring里面
// 添加扫描路径，告知接口在什么地方
@MapperScan("top/mphy/blog/springbootapi/mapper")
public class SpringbootAdminApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootAdminApplication.class, args);
    }
}
```

Entity 实体层配置。  
存放实体，比如用户的基本数据，根据数据库表中的内容设置。
```java
// lombok：简化代码，不用生成getter和setter
import lombok.Data;

@Data
public class User {
    private int id;
    private  String username;
    private  String password;
    private  String nickname;
    private  String age;
    private  String sex;
    private  String address;
}
```

Mapper 层配置。  
- Mapper 层：持久层(==dao层)，主要与数据库进行交互，编写 SQL 代码。
- Mapper 层会调用 Entity 层，Mapper 中会定义实际使用到的方法，比如增删改查。
- Mapper 层的数据源和数据库连接的参数都是在配置文件中进行配置的，配置文件在 application.yml 中
- 数据持久化操作就是指，把数据放到持久化的介质中，同时提供增删改查操作。

```java
@Mapper
public interface UserMapper {
    // 注解 + sql实现查询
    // 查询所有用户信息
    @Select("select * from user")
    List<User> findAll();

    // post请求，添加一条数据
    @Update("INSERT INTO `user` (`name`, `address`, `age`, `sex`, `phone`) " +
            "VALUES (#{name}, #{address}, #{age}, #{sex}, #{phone});")
    // 事务注解
    @Transactional
    void save(User user);
}
```

Service 层：业务层，控制业务。  
- Service 层主要负责业务模块的逻辑应用设计。先设计放接口的类，再创建实现的类。
- Service 层调用 Mapper 层接口，接收 Mapper 层返回的数据，完成项目的基本功能设计。


Controller 层：控制层，控制业务逻辑。
- Controller 层负责具体的业务模块流程的控制，
- Controller 层负责前后端交互，接受前端请求，调用 Service 层，接收 service 层返回的数据，最后返回具体的页面和数据到客户端。
- Controller 层 提供对外接口，调用 Mapper 中的查询等操作
- `@RestController`：所有查询的数据都会被渲染成 JSON。

```java
@RestController
@RequestMapping("/user")
public class UserController {
    // 相关逻辑代码
}
```

### 1.6 后端设置允许跨域请求

`http://localhost:8081` 为允许跨域请求的客户端URL。

```java
@CrossOrigin(origins = "http://localhost:8081", maxAge = 3600)
@RestController
@RequestMapping("/login")
public class UserController {
    // 后端业务代码
}
```

### 1.7 RESTfull 接口设计原则

> 参考官网：<a href="https://restfulapi.cn/">https://restfulapi.cn/</a>

接口设计原则：请求 = 动词 + 宾语  
增删改查：

- 增：`POST`
- 删：`DELETE`
- 改：`PUT`
- 查：`GET`

### 1.8 Mybatis 动态 SQL 编写

> 需求：更新部分用户的数据，前端没有提供某些参数时，不更新数据库中表的字段。

要在带注解的映射器接口类中使用动态 SQL，可以使用 script 元素。比如:

```java
@Update({"<script>",
        "update Author",
        "  <set>",
        "    <if test='username != null'>username=#{username},</if>",
        "    <if test='password != null'>password=#{password},</if>",
        "    <if test='email != null'>email=#{email},</if>",
        "    <if test='bio != null'>bio=#{bio}</if>",
        "  </set>",
        "where id=#{id}",
        "</script>"})
void updateAuthorValues(Author author);
```



## 2. Vue.js

### 2.1 使用 Axios

Vue.js 项目中使用 Axios 步骤：
1. 下载依赖
  ```
  npm install axios --save
  ```
2. 在 main.js 中引入依赖并全局注册
  ```js
  import axios from 'axios'
  Vue.prototype.$axios = axios
  ```
  在组件中使用：
  ```js
  methods: {
    save() {
      this.$axios
        .post('/api/user', this.form)
        .then((res) => {
          console.log(res)
        })
        .catch((err) => {
          console.log(err)
        })
    }
  }
  ```

### 2.2 前端设置跨域

```js
// 跨域配置
module.exports = {
  // 记住，别写错了devServer
  devServer: {      
    // 设置本地默认端口  选填
    // port: 9876,
    // 设置代理，必须填
    proxy: {
      // 设置拦截器  拦截器格式   斜杠+拦截器名字，名字可以自己定       
      '/api': {
        // 代理的目标地址
        target: 'http://localhost:8081',
        // 是否设置同源，输入是的
        changeOrigin: true,
        // 路径重写
        pathRewrite: {
          // 选择忽略拦截器里面的单词
          '/api': ''
        }
      }
    }
  }
}

```