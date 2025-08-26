# MyBatis-Plus

版本升级：3.1.0---->3.5.7

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
    <version>${mybatis-plus-starter.version}</version>
</dependency>
```



```java
// 旧版本写法
query.setAsc("id");
// 新版本写法
query.addOrder(OrderItem.asc("id"));
```

`IdType.UUID` 改为 `IdType.ASSIGN_UUID`

`strategy` 改为 `updateStrategy`

## 为何 MyBatis Plus 不支持复合主键，强制唯一 ID 问题解答

MyBatis Plus 不支持复合主键并强制使用唯一的 ID，这是出于以下考虑：

1. 增加了表与表之间的相互依赖性：使用复合主键会使表与表之间的关系更加复杂，增加了维护和管理的难度。
2. 增加了数据复杂的约束、规则：复合主键会增加数据的约束和规则，例如需要约束唯一性，而完全可以使用联合索引来实现。
3. 增加了更新数据的限制：在更新数据时，需要更新所有复合主键的值，这增加了更新操作的限制和复杂性。
4. 严重的数据冗余和更新异常问题：复合主键可能导致数据冗余和更新异常的问题，特别是在大型系统中，可能会出现更新异常的情况。
5. 性能问题：使用复合主键时，查询某个 ID 时无法使用索引，会导致性能下降。

综上所述，虽然使用复合主键可以省去一个 ID 字段，但是这种做法的缺点大于优点，不建议也不推荐这样做。MyBatis Plus 坚持使用唯一的 ID，以保证数据管理的简单性、可维护性和性能。

## 项目

```xml
# 等价于 menu_id != #{menuId}
menu_id <![CDATA[ <> ]]> #{menuId}
```

- **避免XML解析错误**：使用`CDATA`段可以确保XML解析器正确处理SQL语句中的特殊字符。
- **SQL标准**：`<>`是SQL标准中推荐的不等于操作符，使用它可以确保SQL语句的兼容性。
- **兼容性**：虽然`!=`在大多数SQL环境中有效，但使用`<>`可以避免潜在的兼容性问题。

```java
# updateStrategy = FieldStrategy.IGNORED：表示在更新操作时，如果该字段的值为 null，则不会更新数据库中的对应列。
@TableField(value = "weight", updateStrategy = FieldStrategy.IGNORED)
private String weight;
```

