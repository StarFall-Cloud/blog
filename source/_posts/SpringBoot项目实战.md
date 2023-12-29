---
title: SpringBoot项目实战
date: 2023-12-28 20:56:47
tags: SpringBoot

---



##### SpringBoot查询数据

1、pom.xml 引入 mybatis 依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>3.0.2</version>
    </dependency>
</dependencies>
```

2、entity包 -> User 实体类

```java
package com.demo.springbootfirst.entity;

import lombok.Data;

// Data注解，代替 Getter和 Setter 方法
@Data
public class User {
    private Integer id;
    private String username;
    private String password;
    private String nickname;
    private String email;
    private String phone;
    private String address;
}
```

3、mapper包 -> UserMapper 接口

```java
package com.demo.springbootfirst.mapper;

import com.demo.springbootfirst.entity.User;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;

import java.util.List;

@Mapper
public interface UserMapper {

    @Select("select * from sys_user")
    List<User> findAll();
}
```

4、controller包 -> UserController 类

```java
package com.demo.springbootfirst.controller;

import com.demo.springbootfirst.entity.User;
import com.demo.springbootfirst.mapper.UserMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
public class UserController {

    @Autowired
    private UserMapper userMapper;

    // GetMapping注解 依赖于 @RestController 注解实现数据查询
    @GetMapping("/")
    public List<User> index() {
        return userMapper.findAll();
    }
}
```

5、SpringbootFirstApplication 测试类

```java
package com.demo.springbootfirst;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RestController;

@RestController
@SpringBootApplication
public class SpringbootFirstApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootFirstApplication.class, args);
    }
}
```





#### 前端跨域问题

![image-20231009154753421](SpringBoot%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98/image-20231009154753421.png)



#### Springboot跨域设置

```java
package com.demo.springbootfirst.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

@Configuration
public class CorsConfig {

    // 当前跨域请求最大有效时长。这里默认1天
    private static final long MAX_AGE = 24 * 60 * 60;

    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("http://localhost/"); // 1 设置访问源地址
        corsConfiguration.addAllowedHeader("*"); // 2 设置访问源请求头
        corsConfiguration.addAllowedMethod("*"); // 3 设置访问源请求方法
        corsConfiguration.setMaxAge(MAX_AGE);
        source.registerCorsConfiguration("/**", corsConfiguration); // 4 对接口配置跨域设置
        return new CorsFilter(source);
    }
}
```



###### 忽略某个字段，不展示给前端

```java
@JsonIgnore
```



#### MyBatis-Plus

实现多条件分页查询。

mybatis-plus 依赖

```xml
<!-- Mybatis-plus -->
<dependencies>
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.5.2</version>
    </dependency>
</dependencies>
```



MybatisPlusConfig.java

```java
package com.demo.springbootfirst.config;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@MapperScan("com.demo.springbootfirst.mapper")
public class MybatisPlusConfig {
    /**
     * 添加分页插件
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        //如果配置多个插件,切记分页最后添加
        return interceptor;
    }
}
```



##### 错误示例

![image-20231009184817359](SpringBoot%E9%A1%B9%E7%9B%AE%E5%AE%9E%E6%88%98/image-20231009184817359.png)



```java
package com.demo.springbootfirst.controller;

import com.demo.springbootfirst.entity.User;
import com.demo.springbootfirst.mapper.UserMapper;
import com.demo.springbootfirst.service.UserService;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.tags.Tag;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Slf4j
@RestController
@RequestMapping("/user")
@Schema(name = "", description = "")
@Tag(name = "JsonController tags")
public class UserController {
    @Autowired
    private UserMapper userMapper;

    @Autowired
    private UserService userService;

    // 新增和修改
    @PostMapping
    public boolean save(@RequestBody User user) {
        // 新增或者更新
//        return userService.save(user);
        return userService.saveUser(user);
    }

    // 查询所有数据
    @GetMapping
    public List<User> findAll() {
        List<User> all = userMapper.findAll();
        return all;
    }

    @DeleteMapping("/{id}")
    public Integer delete(@PathVariable Integer id) {
        return userMapper.deleteById(id);
    }

    // 分页查询
    // 接口路径：/user/page?pageNum=1&pageSize=10
    // @RequestParam接收
    // limit第一个参数 = (pageNum - 1) * pageSize
    // pageSize
    @GetMapping("/page")
    public Map<String, Object> findPage(@RequestParam Integer pageNum,
                                        @RequestParam Integer pageSize,
                                        @RequestParam String username) {
        pageNum = (pageNum - 1) * pageSize;
        username = "%" + username + "%";
        List<User> data = userMapper.selectPage(pageNum, pageSize, username);
        Integer total = userMapper.selectTotal(username);
        Map<String, Object> res = new HashMap<>();
        res.put("data", data);
        res.put("total", total);
        return res;
    }
}
```



```java
package com.demo.springbootfirst.controller;

import com.demo.springbootfirst.entity.User;
import com.demo.springbootfirst.mapper.UserMapper;
import com.demo.springbootfirst.service.UserService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

@RestController
@RequestMapping(value = "/user")
@Tag(name = "user-controller", description = "描述")
public class UserController {

    @Autowired
    private UserMapper userMapper;

    @Autowired
    private UserService userService;

    // 新增和修改
    @PostMapping
    public boolean save(@RequestBody User user) {
        // 新增或者更新
//        return userService.save(user);
        return userService.saveUser(user);
    }

    // 查询所有数据
//    @GetMapping("/user")
    @ResponseBody
    @Operation(summary = "获取用户信息", description = "获取用户信息")
    public List<User> findAll() {
        List<User> all = userMapper.findAll();
        return all;
    }

//    @DeleteMapping("/{id}")
    public Integer delete(@PathVariable Integer id) {
        return userMapper.deleteById(id);
    }

    // 分页查询
    // 接口路径：/user/page?pageNum=1&pageSize=10
    // @RequestParam接收
    // limit第一个参数 = (pageNum - 1) * pageSize
    // pageSize
//    @GetMapping("/page")
    public Map<String, Object> findPage(@RequestParam Integer pageNum,
                                        @RequestParam Integer pageSize,
                                        @RequestParam String username) {
        pageNum = (pageNum - 1) * pageSize;
        username = "%" + username + "%";
        List<User> data = userMapper.selectPage(pageNum, pageSize, username);
        Integer total = userMapper.selectTotal(username);
        Map<String, Object> res = new HashMap<>();
        res.put("data", data);
        res.put("total", total);
        return res;
    }

//    @PostMapping()
//    public void addProduct() {
//        System.out.println("整合swagger嘻嘻！");
//    }
}
```



#### 集成 Swagger-UI

访问地址： http://localhost:8080/swagger-ui/index.html



###### Swagger

SwaggerConfig.java

```java
package com.demo.springbootfirst.config;

import io.swagger.v3.oas.models.ExternalDocumentation;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import org.springdoc.core.models.GroupedOpenApi;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SwaggerConfig {

    @Bean
    public OpenAPI springShopOpenAPI() {
        return new OpenAPI()
                .info(new Info().title("项目平台接口文档")
                        .description("接口描述")
                        .version("2.0.0"))
                .externalDocs(new ExternalDocumentation().description("")
                        .url(""));
    }

    @Bean
    public GroupedOpenApi adminApi() {
        return GroupedOpenApi.builder()
                //分组名
                .group("admin")
                //扫描路径，将路径下有swagger注解的接口解析到文档中
                .packagesToScan("com.demo.springbootfirst.controller")
                .build();
    }
}
```

pom.xml 依赖

```xml
<dependencies>
        <!-- Springdoc -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.1.0</version>
        </dependency>
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-common</artifactId>
            <version>2.1.0</version>
        </dependency>
</dependencies>
```



##### vuex安装

npm i vuex@3(vue需要指定安装vuex@3版本) -S

store.js

```js
import Vue from "vue";
import Vuex from "vuex"

Vue.use(Vuex)

const store = new Vuex.Store({
    state:{
        currentPathName: ''
    },
    mutations: {
        setPath(state) {
            state.currentPathName = localStorage.getItem("currentPathName")
        }
    }
})

export default store
```

引入vuex

```js
import store from './store';

new Vue({
  router,
  store,
  render: h => h(App)
}).$mount('#app')
```

router/index.js

```js
import Vue from 'vue'
import VueRouter from 'vue-router'
import store from '@/store'

Vue.use(VueRouter)

const routes = [
  {
    path: '/',
    component: () => import('../views/Manager.vue'),
    redirect: "/home",
    children: [
      { path: 'home', name: '首页', component: () => import('../views/Home.vue')},
      { path: 'user', name: '用户管理', component: () => import('../views/User.vue')}
    ]
  },
  {
    path: '/about',
    name: 'About',
    component: () => import('../views/About.vue')
  }
]

const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes
})

// 路由守卫
router.beforeEach((to, from, next) => {
  localStorage.setItem("currentPathName", to.name) // 设置当前的路由名称，为了在Header组件中使用
  store.commit("setPath") // 触发store的数据更新
  next() // 放行路由
})

export default router
```

Header.vue

```vue
  computed: {
    currentPathName() {
      return this.$store.state.currentPathName; // 需要监听的数据
    }
  },
  watch: { // 监听路由变化
    currentPathName(newVal, oldVal) {
      console.log(newVal);
    }
  }
// 使用
      <el-breadcrumb separator="/" style="display: inline-block; margin-left: 10px;">
        <el-breadcrumb-item :to="'/'">首页</el-breadcrumb-item>
        <el-breadcrumb-item>{{ currentPathName }}</el-breadcrumb-item>
      </el-breadcrumb>
```



Login.vue

```vue
<template>
  <div class="wrapper">
    <div style="margin: 200px auto; background-color: #fff; width: 350px; height: 300px; padding: 20px; border-radius: 10px;">
      <div style="margin: 20px; text-align: center; font-size: 24px;"><b>登录</b></div>
      <el-input size="medium" style="margin: 10px 0" prefix-icon="el-icon-user" v-model="user.username"></el-input>
      <el-input size="medium" style="margin: 10px 0" prefix-icon="el-icon-lock" show-password v-model="user.password"></el-input>
      <div style="margin: 10px 0; text-align: right;">
        <el-button type="primary" size="small" autocomplete="off">登录</el-button>
        <el-button type="waring" size="small" autocomplete="off">注册</el-button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Login',
  data() {
    return {
      user: {}
    }
  }
}
</script>

<style>
.wrapper {
  height: 100vh;
  background-image: linear-gradient(to bottom right, #fc466b, #3f5efb);
  overflow: hidden;
}
</style>
```



```vue
<template>
  <div class="login">
    <div class="mylogin" align="center">
      <h4>登录</h4>
      <el-form :model="loginForm" :rules="loginRules" ref="loginForm" label-width="0px">
        <el-form-item label="" prop="account" style="margin-top: 20px">
          <el-row>
            <el-col :span="2">
              <span class="el-icon-s-custom"></span>
            </el-col>
            <el-col :span="22">
              <el-input class="inps" placeholder="账号" v-model="loginForm.account">
              </el-input>
            </el-col>
          </el-row>
        </el-form-item>
        <el-form-item label="" prop="passWord">
          <el-row>
            <el-col :span="2">
              <span class="el-icon-lock"></span>
            </el-col>
            <el-col :span="22">
              <el-input class="inps" type="password" placeholder="密码" v-model="loginForm.passWord"></el-input>
            </el-col>
          </el-row>
        </el-form-item>
        <el-form-item style="margin-top: 30px">
          <el-button type="primary" round class="submitBtn" @click="submitForm">登录
          </el-button>
        </el-form-item>
        <div class="unlogin">
          <!-- <router-link :to="{ path: '/forgetpwd' }"> 忘记密码? </router-link>
          | -->
          <router-link :to="{ path: '/register' }">
            <a href="register.vue" target="_blank" align="right">注册新账号</a>
          </router-link>
        </div>
      </el-form>
    </div>
  </div>
</template>

<script>
import { mapMutations } from "vuex";

export default {
  name: "Login",
  data: function () {
    return {
      loginForm: {
        account: "",
        passWord: "",
      },
      loginRules: {
        account: [{ required: true, message: "请输入账号", trigger: "blur" }],
        passWord: [{ required: true, message: "请输入密码", trigger: "blur" }],
      },
    };
  },

  methods: {
    ...mapMutations(["changeLogin"]),
    submitForm() {
      const userAccount = this.loginForm.account;
      const userPassword = this.loginForm.passWord;
      if (!userAccount) {
        return this.$message({
          type: "error",
          message: "账号不能为空！",
        });
      }
      if (!userPassword) {
        return this.$message({
          type: "error",
          message: "密码不能为空！",
        });
      }
      console.log("用户输入的账号为：", userAccount);
      console.log("用户输入的密码为：", userPassword);

    },
  },
};
</script>

<style>
.login {
  width: 100vw;
  padding: 0;
  margin: 0;
  height: 100vh;
  font-size: 20px;
  background-position: left top;
  background-color: #242645;
  color: #fff;
  font-family: "Source Sans Pro";
  position: relative;
}

.mylogin {
  width: 360px;
  height: 320px;
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  margin: auto;
  padding: 50px 40px 40px 40px;
  box-shadow: -15px 15px 15px rgba(6, 17, 47, 0.7);
  opacity: 1;
  background: linear-gradient(
    230deg,
    rgba(53, 57, 74, 0) 0%,
    rgb(0, 0, 0) 100%
  );
}

.inps input {
  border: none;
  color: #fff;
  background-color: transparent;
  font-size: 16px;
}

.submitBtn {
  background-color: transparent;
  color: #39f;
  width: 200px;
}
</style>
```



















