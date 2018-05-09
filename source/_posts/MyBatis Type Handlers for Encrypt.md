---

title: MyBatis Type Handlers for Encrypt
date: 2017-04-17
updated: 2017-05-23
description: MyBatis 数据库加密工具
tags:
  - MyBatis
categories:
  - 最佳实践
---

## 背景

应公司安全部门要求，需要对数据库中的敏感信息做加密处理。由于此次需求涉及的字段较多，手动加解密颇为不便且改动较大，一个更加简单、通用的解决方案势在必行。

`typehandlers-encrypt` 项目在这种背景下诞生，用户使用时无需更改业务代码，仅需少量配置即可实现数据库指定字段的加解密操作，大大减小了对用户的影响。


## 实现原理

`typehandlers-encrypt` 基于 MyBatis 的 TypeHandler 开发，通过 TypeHandler 可以在 JavaType 和 JdbcType 中互相转换的特性，拦截 JavaType 为 `com.github.trang.typehandlers.alias.Encrypt` 的 SQL，在预处理语句（PreparedStatement）中设置参数时自动加密，并在结果集（ResultSet）中取值时自动解密。

注：由于依赖 MyBatis，使用时需要将 `EncryptTypeHandler` 和 `Encrypt` 注册到 MyBatis，否则无法生效，注册方式见 **声明 EncryptTypeHandler**。


## 应用
### 依赖
```xml
<dependency>
    <groupId>com.github.drtrang</groupId>
    <artifactId>typehandlers-encrypt</artifactId>
    <version>1.0.2</version>
</dependency>
```

### 声明 EncryptTypeHandler
#### 1. 单独使用 MyBatis
```xml
<!-- mybatis-config.xml -->
<typeAliases>
    <package name="com.github.trang.typehandlers.alias" />
</typeAliases>

<typeHandlers>
    <package name="com.github.trang.typehandlers.type" />
</typeHandlers>
```

#### 2. 与 Spring 结合
```java
@Bean
public SqlSessionFactory sqlSessionFactory(Configuration config) {
    SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
    factory.setTypeAliasesPackage("com.github.trang.typehandlers.alias");
    factory.setTypeHandlersPackage("com.github.trang.typehandlers.type");
    return factory.getObject();
}
```

#### 3. 与 SpringBoot 结合
```yaml
##application.yml
mybatis:
    type-aliases-package: com.github.trang.typehandlers.alias
    type-handlers-package: com.github.trang.typehandlers.type
```

注：以上配置方式**任选其一**即可，请根据实际情况选择。

### 使用 EncryptTypeHandler
```xml
<!-- select -->
<resultMap id="BaseResultMap" type="user">
    <id column="id" property="id" jdbcType="BIGINT" />
    <result column="username" javaType="string" jdbcType="VARCHAR" property="username" />
    <result column="password" javaType="encrypt" jdbcType="VARCHAR" property="password" />
</resultMap>

<!-- insert -->
<insert id="insert" parameterType="user">
    insert into user (id, username, password)
    values (#{id,jdbcType=BIGINT}, #{username,jdbcType=VARCHAR}, #{password, javaType=encrypt, jdbcType=VARCHAR})
</insert>

<!-- update -->
<update id="update" parameterType="user">
    update user set password=#{password, javaType=encrypt, jdbcType=VARCHAR} where id=#{id}
</update>
```


## 进阶
`typehandlers-encrypt` 内置了 16 位长度密钥与 AES 加密算法，支持开箱即用，但用户也可以自定义密钥和加密算法，只需要在配置文件中声明对应的属性即可。需要注意，**两者同时配置时，要声明在同一文件里**。

配置示例：
```properties
encrypt.private.key=xxx
encrypt.class.name=com.github.trang.typehandlers.crypt.SimpleEncrypt
```

### 配置文件查找
方式一：项目启动时，会在项目的 Classpath 中依次查找名称为 `encrypt`、`properties/config-common`、`properties/config`、`config`、`application` 的 Properties 文件，直到文件存在且文件中包含名称为 `encrypt.private.key` 的属性时停止。

方式二：如果项目中不存在以上文件，且不想单独新增，也可以在项目启动时调用 `ConfigUtil.bundleNames("xxx")` 来指定要读取的文件，这时只会从用户给定的文件中查找。

**当没有查找到相应配置时，项目会使用内置的配置。**

### 自定义密钥
`typehandlers-encrypt` 支持自定义密钥，只需在配置文件中声明即可。
```properties
encrypt.private.key=xxx
```

### 自定义加密算法
`typehandlers-encrypt` 默认的加密算法是 **AES 对称加密**，如果默认算法不满足实际需求，用户可以自己实现 `com.github.trang.typehandlers.crypt.Crypt` 接口，并在配置文件中声明实现类的全路径。
```properties
encrypt.class.name=com.github.trang.typehandlers.crypt.SimpleEncrypt
```


## 演示
目前项目已开源，并上传到 [Github](https://github.com/drtrang/typehandlers-encrypt)，大家感兴趣的话可以阅读源码。[Github](https://github.com/drtrang/typehandlers-encrypt) 中有配套的 Demo 演示 [`typehandlers-encrypt-demo`](https://github.com/drtrang/typehandlers-encrypt-demo)，其中包括 `typehandlers-encrypt` 完整的使用方式，可以作为参考。


## About Me
QQ：349096849
Email：donghao.l@hotmail.com
Blog：[Trang's Blog](http://blog.trang.space)


> **注意：**
> 1. 目前 `EncryptTypeHandler` 只支持 JavaType 为 **String** 的情形，如有其它需求，请及时联系我。
> 2. 加密时字段只过滤 **null 值**，非 null 的字段不做任何处理直接加密。
