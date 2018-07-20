
---
title: "SpringBoot整合mybatis"
date: 2017-12-22 11:42:54
categories: SoringBoot
tags: 
	- SpringBoot,
	- mybatis
---

**SpringBoot整合mybatis,测试简单的CURD**
<!-- more -->


### 前言
- 整合springboot和mybatis仅仅为了学习和简单的使用

### 导入依赖
- 在`pom.xml`中引入依赖,如下:
```xml
<!--引入mybatis,因为带有JDBC所以不需要引入JDBC了-->
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
	<version>1.2.0</version>
</dependency>
```

### 实体类
- **这里实体类,我是用的测试JPA时的实体类,也可以按照平常的写法,先private字段,在get,set也可以**
```java
package com.example.demo.model;

import javax.persistence.*;
import java.io.Serializable;

@Entity
@Table(name="user")
public class SystemUser implements Serializable{
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY) //id自增
	@Column(name = "id")
	private Integer id;
	
	@Column(name="user_name")
	private String name;
	
	@Column(name="user_password")
	private String password;

	public SystemUser(String name, String password, Integer id) {
		this.name = name;
		this.password = password;
		this.id = id;
	}

	public SystemUser(String name, String password) {
		this.name = name;
		this.password = password;
	}

	@Override
	public String toString() {
		return "SystemUser{" +
				"id=" + id +
				", name='" + name + '\'' +
				", password='" + password + '\'' +
				'}';
	}

	public SystemUser() {
	}

	public Integer getId() {
		return id;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}
	
	
}

```

### 编写Dao层

```java
package com.example.mybatisDeno.mapper;

import com.example.demo.model.SystemUser;
import org.apache.ibatis.annotations.*;

import java.util.List;

/**
 * 创建时间:2017/12/22 0022
 * 创建人:lq
 * 使用注解进行简单的增删改查
 */
@Mapper //@Mapper将UserDao声明为一个Mapper接口
public interface UserDao {
    @Results({
            @Result(property = "id", column = "id"),
            @Result(property = "name", column = "user_name"),
            @Result(property = "password", column = "user_password")
    })

    /**
     * 查询
     */
    @Select("SELECT * FROM user WHERE user_name = #{name}") //3
    List<SystemUser> get(String name);

    /**
     * 增加,返回增加元素的ID
     * @param user SystemUser实体
     * @return 返回增加元素ID
     */
    @Insert("INSERT INTO `user`(user_name,user_password) VALUES(#{name},#{password})")
    @Options(useGeneratedKeys = true, keyColumn = "id", keyProperty = "id")
    Integer add(SystemUser user);


    /**
     * 删除
     * @param id
     * @return 返回影响的行数
     */
    @Delete("Delete from user where id = #{id}")
    Long delete(Integer id);

    /**
     * 修改
     * @param user SystemUser实体
     * @return 返回影响的行数
     */
    @Update("update user set user_name = #{name}, user_password = #{password} where id = #{id}")
    Long update(SystemUser user);

}

```
- `@Mapper` 将UserDao声明为一个Mapper接口
- `@Results` 字段与数据库的映射列表
- `@Result` 进行详细的映射,其中property是User类的属性名，colomn是数据库表的字段名 
- `@Select` 写入查询
- `@Update` 写入更新语句
- `@Delete` 写入删除语句
- `@Insert` 写入插入语句
- `@Options` 设置主键,其中useGeneratedKeys是使用主键,keyProperty实体类中主键的名字,keyColumn数据库中主键的名字

### 编写service层

- 如下;
```java
package com.example.mybatisDeno.service;

import com.example.demo.model.SystemUser;
import com.example.mybatisDeno.mapper.UserDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * 创建时间:2017/12/22 0022
 * 创建人:lq
 */
@Service
public class UserService {
    @Autowired
    private UserDao userDao;

    /**
     * 根据name查询
     * @param name
     * @return
     */
    public List<SystemUser> get(String name){
        return userDao.get(name);
    }

    /**
     * 增加
     * @param name,password
     * @return
     */
    public Integer add(String name, String password){
        SystemUser user = new SystemUser(name,password);
        return userDao.add(user);
    }

    /**
     * 删除
     * @param id
     * @return
     */
    public Long delete(Integer id){
        return userDao.delete(id);
    }

    /**
     * 修改
     * @param name,password
     * @return
     */
    public Long update(String name, String password, Integer id){
        SystemUser user = new SystemUser(name, password, id);
        return userDao.update(user);
    }
}

```

### 编写controller层

```java
package com.example.mybatisDeno.controller;

import com.example.demo.model.SystemUser;
import com.example.mybatisDeno.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

/**
 * 创建时间:2017/12/22 0022
 * 创建人:lq
 */
@RestController
@RequestMapping(value = "/mb")
public class UserController {
    @Autowired
    private UserService userService;

    @RequestMapping(value = "/")
    public String hello(){
        return "mybatis";
    }

    @RequestMapping(value = "/get")
    public List<SystemUser> get(@RequestParam String name){
        return userService.get(name);
    }

    @RequestMapping(value = "/add")
    public Integer add(@RequestParam String name, @RequestParam String password){
        return userService.add(name, password);
    }

    @RequestMapping(value = "/delete")
    public Long delete(@RequestParam Integer id){
        return userService.delete(id);
    }

    @RequestMapping(value = "/update")
    public Long update(@RequestParam String name, @RequestParam String password, @RequestParam Integer id){
        return userService.update(name, password ,id);
    }
}

```

### 总结
- 这里详细说明一下MyBatis注解

**使用@Param**
如下代码:

```java
@Insert("INSERT INTO USER(NAME, AGE) VALUES(#{name}, #{age})")
int insert(@Param("name") String name, @Param("age") Integer age);

```

这种方式很好理解，`@Param`中定义的`name`对应了SQL中的`#{name}`，`age`对应了SQL中的`#{age}`

**使用Map**
如下代码:

```java
@Insert("INSERT INTO USER(NAME, AGE) VALUES(#{name,jdbcType=VARCHAR}, #{age,jdbcType=INTEGER})")
int insertByMap(Map<String, Object> map);
```

- 对于Insert语句中需要的参数，我们只需要在`map`中填入同名的内容即可，具体如下面代码所示：

```java
Map<String, Object> map = new HashMap<>();
map.put("name", "CCC");
map.put("age", 40);
userMapper.insertByMap(map);
```

**使用对象**

-  可直接使用普通的Java对象来作为查询条件的传参，比如我们可以直接使用User对象:

```java
@Insert("INSERT INTO USER(NAME, AGE) VALUES(#{name}, #{age})")
int insertByUser(User user);
```






