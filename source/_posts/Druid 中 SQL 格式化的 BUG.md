---

title: Druid 中 SQL 格式化的 BUG
date: 2018-02-26
tags:
  - druid
categories:
  - 技术内幕

---

前段时间在写一个格式化 SQL 的拦截器（[SqlFormatterInterceptor](https://github.com/drtrang/spring-boot-autoconfigure/blob/master/src/main/java/com/github/trang/autoconfigure/mybatis/SqlFormatterInterceptor.java)），其中格式化的部分借用了 Druid 现成的工具类 [SQLUtils](https://github.com/drtrang/spring-boot-autoconfigure/blob/master/src/main/java/com/github/trang/autoconfigure/mybatis/SqlFormatterInterceptor.java#L64)，但在使用过程中发现输出的 SQL 中会出现多余的空格，复现场景如下。

<!-- more -->

### 原始 SQL
```java
String sql = "SELECT id,task_id,task_source, housedel_code, del_type, office_address, company_code, brand, " +
                "class1_code, class2_code, class2_name, status, create_time, end_time, creator_ucid, creator_name, " +
                "org_code, prove_id, prove_time, audit_ucid, audit_name, audit_reject_reason, audit_content, " +
                "audit_time, pass_mode, sms_content, sms_time \r\n FROM " +
                "sh_true_house_task \r\n  WHERE office_address = 0 AND status = 0 ORDER     BY id DESC";
```

### 默认格式化
```java
// 格式化代码
SQLUtils.formatMySql(sql)
// 输出结果
SELECT id, task_id, task_source, housedel_code, del_type
    , office_address, company_code, brand, class1_code, class2_code
    , class2_name, status, create_time, end_time, creator_ucid
    , creator_name, org_code, prove_id, prove_time, audit_ucid
    , audit_name, audit_reject_reason, audit_content, audit_time, pass_mode
    , sms_content, sms_time
FROM sh_true_house_task
WHERE office_address = 0
    AND status = 0
ORDER BY id DESC
```

### 仅大写格式化
```java
// 格式化代码
SQLUtils.formatMySql(sql, new FormatOption(VisitorFeature.OutputUCase))
// 输出结果
SELECT id, task_id, task_source, housedel_code, del_type , office_address, company_code, brand, class1_code, class2_code , class2_name, status, create_time, end_time, creator_ucid , creator_name, org_code, prove_id, prove_time, audit_ucid , audit_name, audit_reject_reason, audit_content, audit_time, pass_mode , sms_content, sms_time FROM sh_true_house_task WHERE office_address = 0 AND status = 0 ORDER BY id DESC
```

由上述结果可以看到，在仅使用 `OutputUCase` 配置时，输出的 SQL 中 `del_type`、`class2_code` 等字符后有多余的空格，经过仔细的翻看源码，问题最终定位在 [SQLASTOutputVisitor.java](https://github.com/alibaba/druid/blob/master/src/main/java/com/alibaba/druid/sql/visitor/SQLASTOutputVisitor.java#L440) 的第 440 行。

这段代码发生在格式化的最后步骤，拼接输出 SQL 的时候，具体逻辑是分析查询的数据库列，每当 column 数量为 5 的倍数时就调用 `println()` 方法。
```java
// SQLASTOutputVisitor.java#L386
if (lineItemCount >= selectListNumberOfLine) {
    lineItemCount = paramCount;
    println();
}
```

而 `println()` 的实现中，如果不开启 `OutputPrettyFormat` 特性，就会在当前 SQL 后增加一个空格。
```java
// SQLASTOutputVisitor.java#L440
public void println() {
    if (!isPrettyFormat()) {
        print(' ');
        return;
    }
}
```

目前尚不明确这行代码的意图，但就结果来看，最终得到的 SQL 并不符合强迫症的审美 : (，该问题我已提到官方 [ISSUE#2347](https://github.com/alibaba/druid/issues/2347)，期待官方给出答案。