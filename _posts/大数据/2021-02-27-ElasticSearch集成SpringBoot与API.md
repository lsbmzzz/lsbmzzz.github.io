---
layout:     post
title:      ElasticSearch集成SpringBoot与API
subtitle:   ElasticSearch集成SpringBoot与API
date:       2021-02-27
author:     张三
header-img: img/ElasticSearch.jpg
catalog: true
mathjax: true
tags:
    - ElasticSearch
    - SpringBoot

---

创建项目时，勾选ElasticSearch依赖

![SpringBoot导入ElasticSearch](/img/大数据/SpringBoot导入ElasticSearch.png)

> 可在properties中自定义版本保证ElasticSearch版本一致

## 导入依赖

```xml
<!--springboot的elasticsearch服务-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

## 注入RestHighLevelClient客户端
```java
@Configuration
public class ElasticSearchClientConfig {
    @Bean
    public RestHighLevelClient restHighLevelClient() {
        return new RestHighLevelClient(
                RestClient.builder(
                        new HttpHost("127.0.0.1", 9200, "http")
                )
        );
    }
}
```

## 测试索引操作

```java
@SpringBootTest
class ElasticdemoApplicationTests {

    @Autowired
    private RestHighLevelClient restHighLevelClient;

    // 测试索引创建
    @Test
    void createIndex() throws IOException {
        CreateIndexRequest request = new CreateIndexRequest("index1");
        CreateIndexResponse response = restHighLevelClient.indices().create(request, RequestOptions.DEFAULT);
        System.out.println("====================");
        System.out.println(response);
    }

    // 测试索引查询
    @Test
    void existIndex() throws IOException {
        GetIndexRequest request = new GetIndexRequest("index1");
        boolean exist = restHighLevelClient.indices().exists(request, RequestOptions.DEFAULT);
        System.out.println("=====================");
        System.out.println(exist);
    }

    // 删除索引
    @Test
    void deleteIndex() throws IOException{
        DeleteIndexRequest request = new DeleteIndexRequest("index1");
        AcknowledgedResponse response = restHighLevelClient.indices().delete(request, RequestOptions.DEFAULT);
        System.out.println("====================");
        System.out.println(response);
    }
}
```

## 测试文档操作

```java
@SpringBootTest
class ElasticdemoApplicationTests {

    @Autowired
    private RestHighLevelClient restHighLevelClient;

    // 测试添加文档
    @Test
    void addDocument() throws IOException {
        User user = new User("zhangsan", 18, "男");
        IndexRequest request = new IndexRequest("zhangsan");
        // 设置一些属性
        request.id("1");
        request.timeout("1s");
        // 转换成json字符串
        request.source(JSON.toJSONString(user), XContentType.JSON);
        IndexResponse response = restHighLevelClient.index(request, RequestOptions.DEFAULT);
        System.out.println("=======================");
        System.out.println(response.toString());
        System.out.println(response.status());
    }

    // 测试文档是否存在
    @Test
    void existDocument() throws IOException {
        // 有没有这个index
        GetRequest request = new GetRequest("zhangsan");
        boolean exist = restHighLevelClient.exists(request, RequestOptions.DEFAULT);
        System.out.println("==================");
        System.out.println(exist);
    }

    // 获取文档
    @Test
    void getDocument() throws IOException {
        GetRequest request = new GetRequest("zhangsan", "1");
        GetResponse response = restHighLevelClient.get(request, RequestOptions.DEFAULT);
        System.out.println("================");
        System.out.println(response.getSourceAsString());
        System.out.println(response);
    }

    // 修改文档
    @Test
    void updateDocument() throws IOException {
        User user = new User("zhangsan", 30, "男");
        UpdateRequest request = new UpdateRequest("zhangsan", "1");
        request.doc(JSON.toJSONString(user), XContentType.JSON);
        UpdateResponse response = restHighLevelClient.update(request, RequestOptions.DEFAULT);
        System.out.println("=======================");
        System.out.println(response);
        System.out.println(response.status());
    }

    // 删除文档
    @Test
    void deleteDocument() throws IOException {
        DeleteRequest request = new DeleteRequest("zhangsan", "1");
        DeleteResponse response = restHighLevelClient.delete(request, RequestOptions.DEFAULT);
        System.out.println("===========================");
        System.out.println(response.status());
    }

    // 批量添加
    @Test
    void allDocuments() throws IOException{
        List<User> userList = new ArrayList<User>();
        userList.add(new User("zhangsan1", 11, "男"));
        userList.add(new User("zhangsan2", 12, "男"));
        userList.add(new User("zhangsan3", 13, "男"));
        userList.add(new User("zhangsan4", 14, "男"));
        userList.add(new User("zhangsan5", 15, "男"));
        userList.add(new User("zhangsan6", 16, "男"));

        BulkRequest request = new BulkRequest();

        for(int i = 0; i < userList.size(); i++) {
            request.add(
                    new IndexRequest("zhangsan").id(Integer.toString(i + 1)).source(
                            JSON.toJSONString(userList.get(i)), XContentType.JSON)
            );
        }
        BulkResponse responses = restHighLevelClient.bulk(request, RequestOptions.DEFAULT);
        System.out.println("=====================");
        System.out.println(responses.hasFailures());
    }

    // 查询文档
    @Test
    void searchDocument() throws IOException{
        SearchRequest request = new SearchRequest("zhangsan");
        // 构建查询条件
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        sourceBuilder.highlighter();
        MatchQueryBuilder matchQueryBuilder = QueryBuilders.matchQuery("username", "zhangsan1");
        // 添加条件
        sourceBuilder.query(matchQueryBuilder);
        sourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));
        request.source(sourceBuilder);
        SearchResponse response = restHighLevelClient.search(request, RequestOptions.DEFAULT);
        System.out.println("===========================");
        for(SearchHit documentFields : response.getHits().getHits()) {
            System.out.println(documentFields.getSourceAsString());
        }
    }
}
```