---

title: 禁止 MyBatis 打印 SQL 的功能
date: 2018-02-26
tags:
  - mybatis
  - druid
categories:
  - 技术内幕

---

## 背景
昨天在整理日志的时候，打算将每次执行的 SQL 打印出来（开发环境），便于快速定位问题。

在已知的两种打印方式中，MyBatis 会将预处理 SQL、参数、返回结果分别打印，而 Druid 可以更细致的对打印内容进行自定义，并且可以直接打印可执行 SQL，于是决定使用 Druid 来打印 SQL。

<!-- more -->

#### MyBatis 打印 SQL
```txt
2018-02-26 14:11:05.772 DEBUG 14164 --- [28620-thread-21] c.l.m.q.d.H.selectCustom             : ==>  Preparing: SELECT id, log_type, business_id, operate_type, operator_ucid, operator_name, operator_ip, operation_reason, created_time, brand, log_content FROM sh_housedel_log WHERE business_id = ? AND operate_type = ? AND operation_reason IN ( ? ) LIMIT ? 
2018-02-26 14:11:05.775 DEBUG 14164 --- [28620-thread-21] c.l.m.q.d.H.selectCustom             : ==> Parameters: 101000000122(Long), u(String), house_maintain_update(String), 10(Integer)
2018-02-26 14:11:05.783 DEBUG 14164 --- [28620-thread-21] c.l.m.q.d.H.selectCustom             : <==      Total: 2
```

#### Druid 打印 SQL
```txt
2018-02-26 14:18:02.100 DEBUG 14848 --- [28620-thread-12] druid.sql.Statement                      : {conn-10001, pstmt-20001} executed. SELECT id, log_type, business_id, operate_type, operator_ucid , operator_name, operator_ip, operation_reason, created_time, brand , log_content FROM sh_housedel_log WHERE business_id = 101000000122 AND operate_type = 'u' AND operation_reason IN ('house_maintain_update') LIMIT 10
```

## 问题分析
开启 Druid 打印 SQL 的功能很简单，我会在文章的最后贴出来，但是在关闭 MyBatis 打印功能的时候却遇到了问题。

观察上述日志可以发现，MyBatis 打印时使用的 loggerName 是以当前项目的包名 `com.lianjia.mls` 作为前缀的（实际是 Mapper.xml 的 namespace），并不是 MyBatis 的包名 `org.mybatis`，而我需要在开启当前项目 DEBUG 日志的同时把 MyBatis 打印的 SQL 去掉，所以之前将 `org.mybatis` 的日志级别设为 INFO 的想法并不能生效。

而且由于 loggerName 的原因，对于打印日志的代码位置也毫无头绪，还好通过全局搜索 `Preparing:` 关键字定位到了 `org.apache.ibatis.logging.jdbc.ConnectionLogger` 类。

![image](https://user-images.githubusercontent.com/13851701/36660069-b00cc882-1b11-11e8-94f2-1995bdf39604.png)

然后一路向上查找调用者。

![image](https://user-images.githubusercontent.com/13851701/36660347-d03ab172-1b12-11e8-9cc2-95b264b0d3a6.png)

![image](https://user-images.githubusercontent.com/13851701/36661102-4d472e5a-1b15-11e8-91e2-e0c7481e4ad0.png)

最终发现该 Logger 是由 `org.apache.ibatis.mapping.MappedStatement` 类创建的，默认值为 `mapperId`，并且通过代码我们可以看到这里有个神奇的配置： `logPrefix`，它可以为 loggerName 增加前缀，也就是说，之前通过配置日志级别来禁止 MyBatis 打印 SQL 的想法是可行的，只需要将 `logPrefix` 的日志级别设为 INFO 即可。

![image](https://user-images.githubusercontent.com/13851701/36660544-7a8d0652-1b13-11e8-86e0-d94726097ae4.png)

## 大功告成
至此该问题已完美解决，既保留了项目的 DEBUG 日志，同时又禁止了 MyBatis 的打印 SQL 功能，而且对 MyBatis 本身的日志配置也不影响，可谓一举三得。

## 打印 SQL 配置
### MyBatis 配置
```yaml
# application.yml

mybatis:
  configuration:
    log-prefix: executableSql.

logger:
  executableSql: debug
```

### Druid 配置
```yaml
# application.yml

logger:
  druid.sql.Statement: debug

## 以下功能由 [druid-spring-boot-starter](https://github.com/drtrang/druid-spring-boot) 提供
spring:
  datasource:
    druid:
      slf4j:
        enabled: true
        statement-log-enabled: true
        statement-sql-format-option:
          upp-case: true
          pretty-format: false
        statement-create-after-log-enabled: false
        statement-parameter-set-log-enabled: false
        statement-executable-sql-log-enable: true
        statement-execute-after-log-enabled: false
        statement-parameter-clear-log-enable: false
        statement-close-after-log-enabled: false
```