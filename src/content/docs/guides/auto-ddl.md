---
title: 自动维护DDL
sidebar:
  order: 10
---

在MyBatis-Plus的`3.5.3+`版本中，引入了一项强大的功能：数据库DDL（数据定义语言）表结构的自动维护。这一功能通过执行SQL脚本来实现数据库模式的初始化和升级，与传统的`flyway`工具相比，它不仅支持分表库，还能够控制代码执行SQL脚本的过程。

## 功能概述

- **自动维护DDL历史**：首次使用时，系统会在数据库中创建一个名为`ddl_history`的表，用于记录每次执行的SQL脚本版本信息。
- **灵活的脚本执行**：支持在不同的数据库之间切换数据源，并动态执行相应的脚本命令。
- **企业级特性**：这一功能被视为企业级的高级特性，并且是开源版本的一部分。

## 注意事项

- 当执行DDL操作时，如果脚本中包含切换数据源以创建数据库的操作，可能会遇到异常。解决方法是，在切换到不同的数据库后，动态执行脚本命令。

## 代码示例

以下是一个使用MyBatis-Plus自动维护DDL的Java组件示例：

```java
@Component
public class MysqlDdl implements IDdl {

    /**
     * 获取要执行的SQL脚本文件列表
     */
    @Override
    public List<String> getSqlFiles() {
        return Arrays.asList(
                "db/tag-schema.sql",
                // 从`3.5.3.2`版本开始，支持执行存储过程。在文件名后追加`#$$`，其中`$$`是自定义的完整SQL分隔符。
                // 存储过程脚本以`END`结尾，并追加分隔符`END;$$`表示脚本结束。
                "db/procedure.sql#$$",
                "D:\\db\\tag-data.sql"
        );
    }
}

// 切换到mysql从库，执行SQL脚本 (开源版本无此功能)
ShardingKey.change("mysqlt2");
ddlScript.run(new StringReader("DELETE FROM user;\n" +
        "INSERT INTO user (id, username, password, sex, email) VALUES\n" +
        "(20, 'Duo', '123456', 0, 'Duo@baomidou.com');"));
```

在这个示例中，我们定义了一个`MysqlDdl`组件，它实现了`IDdl`接口，并提供了要执行的SQL脚本文件列表。通过调用`ShardingKey.change`方法，我们可以切换到mysql的从库，并使用`ddlScript.run`方法执行特定的SQL脚本。

通过这种方式，MyBatis-Plus提供了一个高效且自动化的方式来管理数据库的DDL操作，极大地简化了数据库结构的管理和维护工作。

## 自定义运行器

如果集成了MyBatis-Plus的starter的话，会自动实例化一个 DdlApplicationRunner 实例来执行 DDL 脚本。

执行方式为自动提交事务，且忽略错误继续执行（其他脚本参数见如下）。

如果需要自定义控制，请自行注入一个DdlApplicationRunner实例至容器。

```java
    @Bean
    public DdlApplicationRunner ddlApplicationRunner(List<IDdl> ddlList) {
          DdlApplicationRunner ddlApplicationRunner = new DdlApplicationRunner(ddlList);
        // 下面属性自 3.5.11 开始 ...
        // 设置是否自动提交 默认: true
        ddlApplicationRunner.setAutoCommit(false);
        // 设置脚本遇到错误的处理方式 默认: 忽略错误,打印异常 (如果设置为抛出异常,那会终止下一个sql文件处理)
        ddlApplicationRunner.setDdlScriptErrorHandler(DdlScriptErrorHandler.ThrowsErrorHandler.INSTANCE);
        //是否抛出异常中断下个处理器处理 默认: false
        ddlApplicationRunner.setThrowException(true);
        ddlApplicationRunner.setScriptRunnerConsumer(scriptRunner -> {
            scriptRunner.setLogWriter(null);   // 关闭执行日志打印 默认: System.out
            scriptRunner.setErrorLogWriter(null); // 关闭错误日志打印  默认:System.err
            scriptRunner.setStopOnError(true); // 遇到异常是否停止
            scriptRunner.setRemoveCRs(false); //  是否替换\r\n 为 \n 默认: false
        });
        return ddlApplicationRunner;
    }
```