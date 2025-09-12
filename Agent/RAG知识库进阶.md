# RAG 知识库进阶

#### 基于 PGVector 实现向量存储

PGVect⁠or 是经典数据库 P‌ostgreSQL 的扩展，为 Postgr‎eSQL 提供了存储和‌检索高维向量数据的能力。

为什么选择它来实现向量存⁠储呢？因为很多传统业务都会把数据存储在这种关系‌型数据库中，直接给原有的数据库安装扩展就能实现向量相似度搜索、而不需要额外搞一套向量数据库，‎人力物力成本都很低，所以这种方案很受企业青睐，‌也是目前实现 RAG 的主流方案之一。

首先我们准备⁠ PostgreSQL ‌数据库，并为其添加扩展。有 2 种方式，第一种是‎在自己的本地或服务器安装‌，可以参考下列文章实现：

- [Linux 服务器快速安装 PostgreSQL 15 与 pgvector 向量插件实践](https://cloud.baidu.com/article/3229759)
- [宝塔 PostgreSQL 安装 pgvector 插件实现向量存储](https://blog.csdn.net/qq_29213799/article/details/146277755)

这里由于大⁠家更多的是为了学习‌，我们采用更方便的方式 —— 使用现‎成的云数据库，下面‌我们来实操下~

1）首先打开 [阿里云 PostgreSQL 官网](https://www.aliyun.com/product/rds/postgresql)，开通 Serverless 版本，按用量计费，对于学习来说性价比更高：

![image-20250912130445996](assets/image-20250912130445996.png)

2）开通成功后，进入控制台，先创建账号：

进入插件管理，安装 vector 插件：

3）参考 [Spring AI 官方文档](https://docs.spring.io/spring-ai/reference/api/vectordbs/pgvector.html) 整合 PGVector，先引入依赖，版本号可以在 [Maven 中央仓库](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-starter-vector-store-pgvector) 查找：

```
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-vector-store-pgvector</artifactId>
    <version>1.0.0-M7</version>
</dependency>
```

编写配置，建立数据库连接：

```
spring:
  datasource:
    url: jdbc:postgresql://改为你的公网地址/yu_ai_agent
    username: 改为你的用户名
    password: 改为你的密码
  ai:
    vectorstore:
      pgvector:
        index-type: HNSW
        dimensions: 1536
        distance-type: COSINE_DISTANCE
        max-document-batch-size: 10000 
```

注意，在不确定向量维度的情况下，⁠建议不要指定 dimensions 配置。如果未明确指定，Pg‌VectorStore 将从提供的 EmbeddingModel 中检索维度，维度在表创建时设置为嵌入列。如果更改维度，则必‎须重新创建 Vector_store 表。不过最好提前明确你要‌使用的嵌入维度值，手动建表，更可靠一些。

正常情况下⁠，接下来就可以使用‌自动注入的 VectorStore ‎了，系统会自动创建‌库表：

```java
@Autowired
VectorStore vectorStore;



List<Document> documents = List.of(
    new Document("Spring AI rocks!! Spring AI rocks!! Spring AI rocks!! Spring AI rocks!! Spring AI rocks!!", Map.of("meta1", "meta1")),
    new Document("The World is Big and Salvation Lurks Around the Corner"),
    new Document("You walk forward facing the past and you turn back toward the future.", Map.of("meta2", "meta2")));


vectorStore.add(documents);


List<Document> results = this.vectorStore.similaritySearch(SearchRequest.builder().query("Spring").topK(5).build());
```

但是，这种方式不适合我们现在⁠的项目！因为 VectorStore 依赖 Embedd‌ingModel 对象，咱们之前的学习中同时引入了 Ollama 和 阿里云 Dashscope 的依赖，有两个‎ EmbeddingModel 的 Bean，Sprin‌g 不知道注入哪个，就会报下面这种错误：

4）所以让⁠我们换一种更灵活的‌方式来初始化 VectorStore‎。先引入 3 个依‌赖：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-pgvector-store</artifactId>
    <version>1.0.0-M6</version>
</dependency>
```

然后编写配⁠置类自己构造 Pg‌VectorStore，不用 Sta‎rter 自动注入‌：

```
@Configuration
public class PgVectorVectorStoreConfig {

    @Bean
    public VectorStore pgVectorVectorStore(JdbcTemplate jdbcTemplate, EmbeddingModel dashscopeEmbeddingModel) {
        VectorStore vectorStore = PgVectorStore.builder(jdbcTemplate, dashscopeEmbeddingModel)
                .dimensions(1536)                    
                .distanceType(COSINE_DISTANCE)       
                .indexType(HNSW)                     
                .initializeSchema(true)              
                .schemaName("public")                
                .vectorTableName("vector_store")     
                .maxDocumentBatchSize(10000)         
                .build();
        return vectorStore;
    }
}
```

并且启动类要排除掉自动加载，否则也会报错：

```
@SpringBootApplication(exclude = PgVectorStoreAutoConfiguration.class)
public class YuAiAgentApplication {

    public static void main(String[] args) {
        SpringApplication.run(YuAiAgentApplication.class, args);
    }

}
```

5）编写单元测试类，验证效果：

```java
@SpringBootTest
public class PgVectorVectorStoreConfigTest {

    @Resource
    VectorStore pgVectorVectorStore;

    @Test
    void test() {
        List<Document> documents = List.of(
                new Document("Spring AI rocks!! Spring AI rocks!! Spring AI rocks!! Spring AI rocks!! Spring AI rocks!!", Map.of("meta1", "meta1")),
                new Document("The World is Big and Salvation Lurks Around the Corner"),
                new Document("You walk forward facing the past and you turn back toward the future.", Map.of("meta2", "meta2")));
        
        pgVectorVectorStore.add(documents);
        
        List<Document> results = pgVectorVectorStore.similaritySearch(SearchRequest.builder().query("Spring").topK(5).build());
        Assertions.assertNotNull(results);
    }
}
```

以 Deb⁠ug 模式运行，可‌以看到文档检索成功，并且给出了相似度‎得分：