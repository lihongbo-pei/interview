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

### #{} 和 ${} 的区别是什么？

> 科大讯飞

- `${}`是 Properties 文件中的**变量占位符**，它可以用于标签属性值和 sql 内部，属于原样文本替换，可以替换任意内容，比如${driver}会被原样替换为`com.mysql.jdbc. Driver`。

一个示例：根据参数按任意字段排序：

```sql
select * from users order by ${orderCols}
```

`orderCols`可以是 `name`、`name desc`、`name,sex asc`等，实现灵活的排序。

- `#{}`是 sql 的**参数占位符**，MyBatis 会将 sql 中的`#{}`替换为? 号，在 sql 执行前会使用 PreparedStatement 的参数设置方法，按序给 sql 的? 号占位符设置参数值，比如 ps.setInt(0, parameterValue)，`#{item.name}` 的取值方式为使用反射从参数对象中获取 item 对象的 name 属性值，相当于 `param.getItem().getName()`。

> `#{}` 是安全的，它可以有效防止 SQL 注入。

### MyBatis Plus ，怎么做批量的插入？

1. 使用 `saveBatch` 方法

   ```java
   List<User> userList = new ArrayList<>();
   userList.add(new User("Alice", 25));
   userList.add(new User("Bob", 30));
   userService.saveBatch(userList);
   ```

   

2. 使用 `insertBatchSomeColumn` 方法

   `insertBatchSomeColumn`方法可以实现一次性批量插入，并且可以选择性地插入部分字段，优化数据传输。

3. 自定义Mapper实现批量插入

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

