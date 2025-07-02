---
title: Multi-DataSource Support
sidebar:
  order: 14
---

As project scales expand, a single data source can no longer meet complex business requirements, giving rise to multi-data-source (dynamic data source) solutions. This document introduces two MyBatis-Plus multi-data-source extension plugins: the open-source ecosystem's `dynamic-datasource` and the enterprise-grade ecosystem's `mybatis-mate`.

## dynamic-datasource

`dynamic-datasource` is an open-source Spring Boot multi-data-source starter that provides rich features, including data source grouping, sensitive information encryption, independent schema initialization, and more.

### Features

- **Data Source Grouping**: Suitable for various scenarios like read-write separation and one-master-multiple-slaves.
- **Sensitive Information Encryption**: Encrypts database configuration using `ENC()`.
- **Independent Initialization**: Supports independent schema and database initialization for each data source.
- **Custom Annotations**: Supports custom annotations by extending `DS`.
- **Simplified Integration**: Offers quick integration with connection pools like Druid and HikariCP.
- **Component Integration**: Provides integration solutions for components like Mybatis-Plus and Quartz.
- **Dynamic Data Sources**: Supports adding or removing data sources dynamically after project startup.
- **Distributed Transactions**: Offers a distributed transaction solution based on Seata.

### Conventions

- This framework focuses on data source switching and does not restrict specific operations.
- In configuration files, the prefix before an underscore `_` in a data source name represents the group name.
- Data source switching can use either the group name or the specific data source name.
- The default data source name is `master`, which can be modified via `spring.datasource.dynamic.primary`.
- Method-level annotations take precedence over class-level annotations.

### Usage

1. **Add Dependency**:
  - SpringBoot2
  ```xml
  <dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
    <version>${version}</version>
  </dependency>
  ```

 - SpringBoot3
 ```xml
 <dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>dynamic-datasource-spring-boot3-starter</artifactId>
  <version>${version}</version>
 </dependency>
 ```  

2. **Configure Data Sources**:

```yaml
spring:
  datasource:
    dynamic:
      primary: master
      strict: false
      datasource:
        master:
          url: jdbc:mysql://xx.xx.xx.xx:3306/dynamic
          username: root
          password: 123456
          driver-class-name: com.mysql.jdbc.Driver
        slave_1:
          url: jdbc:mysql://xx.xx.xx.xx:3307/dynamic
          username: root
          password: 123456
          driver-class-name: com.mysql.jdbc.Driver
        slave_2:
          url: ENC(xxxxx)
          username: ENC(xxxxx)
          password: ENC(xxxxx)
          driver-class-name: com.mysql.jdbc.Driver
```

3. **Use `@DS` to Switch Data Sources**:

```java
@Service
@DS("slave")
public class UserServiceImpl implements UserService {

  @Autowired
  private JdbcTemplate jdbcTemplate;

  @Override
  @DS("slave_1")
  public List selectByCondition() {
    return jdbcTemplate.queryForList("select * from user where age >10");
  }
}
```

For more tutorials, refer to the [Dynamic-Datasource Official Website](https://github.com/baomidou/dynamic-datasource)

## mybatis-mate

`mybatis-mate` is a paid enterprise component for MyBatis-Plus, featuring many advanced capabilities, including a multi-data-source extension component that provides efficient and simple multi-data-source support.

### Features

- **`@Sharding` Annotation**: Supports data source switching via annotations.
- **Configuration**: Offers flexible data source configuration.
- **Dynamic Loading/Unloading**: Supports dynamic loading and unloading of data sources.
- **Multi-DataSource Transactions**: Supports JTA Atomikos distributed transactions.

### Usage

1. **Configure Data Sources**:

```xml
mybatis-mate:
  sharding:
    primary: mysql
    datasource:
      mysql:
        - key: node1
          ...
        - key: node2
          cluster: slave
          ...
      postgres:
        - key: node1
          ...
```

2. **Use `@Sharding` to Switch Data Sources**:

```java
@Mapper
@Sharding("mysql")
public interface UserMapper extends BaseMapper<User> {

    @Sharding("postgres")
    Long selectByUsername(String username);

}
```

3. **Switch to a Specific Database Node**:

```java
// Switch to mysql slave node2
ShardingKey.change("mysqlnode2");
```

For more examples, refer to:

- Dynamic Loading/Unloading of Multi-Data Sources: 👉 [mybatis-mate-sharding-dynamic](https://gitee.com/baomidou/mybatis-mate-examples/tree/master/mybatis-mate-sharding-dynamic)

- Multi-DataSource Transactions (JTA Atomikos): 👉 [mybatis-mate-sharding-jta-atomikos](https://gitee.com/baomidou/mybatis-mate-examples/tree/master/mybatis-mate-sharding-jta-atomikos)

Through the above introduction, we can see that both `dynamic-datasource` and `mybatis-mate` provide powerful multi-data-source support. Developers can choose the appropriate plugin based on project requirements to achieve flexible data source management.
