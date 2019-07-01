

### Spring Boot 整合 ElasticSearch 入门

​	利用 spring data elasticsearch 模块来对 elasticsearch 进行基础的增删改查，elasticsearch 利用 docker 的方式启动运行。

#### 启动 elasticsearch 

###### 下载 elasticsearch 的 docker 镜像：

```zsh
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.7.2
```

###### 以开发模式运行 elasticsearch 的 docker 容器：

``` zsh
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.7.2
```

###### 浏览器访问 http://localhost:9200/ 出现如下信息就说明启动成功：

``` json
{
  name: "-jrZGj7",
  cluster_name: "docker-cluster",
  cluster_uuid: "-3-MZwe_QImewxAtCSweKQ",
  version: {
    number: "6.7.2",
    build_flavor: "default",
    build_type: "docker",
    build_hash: "56c6e48",
    build_date: "2019-04-29T09:05:50.290371Z",
    build_snapshot: false,
    lucene_version: "7.7.0",
    minimum_wire_compatibility_version: "5.6.0",
    minimum_index_compatibility_version: "5.0.0"
  },
	tagline: "You Know, for Search"
}
```

### 编写测试程序来连接 elasticsearch

###### 创建名为 spring-boot-elasticsearch 的 Spring Boot 项目，在 pom.xml 中添加下面的依赖：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!-- elasticsearch jar -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

###### 在 application.xml 文件中加入下面配置：

``` properties
server.port= 8080
spring.application.name = spring-boot-elasticsearch
# 注意，这个位置一定要配置，不然会报  not avaliable node 的错误的，具体的 cluster-name 是从浏览器中访问 http://localhost:9200/ 接口中获取的
spring.data.elasticsearch.cluster-name=docker-cluster
spring.data.elasticsearch.cluster-nodes=127.0.0.1:9300
```

###### 创建 GoodsIznfo.java 类

```java
package com.ikang.springbootelasticsearch.document;
import org.springframework.data.elasticsearch.annotations.Document;

import java.io.Serializable;

@Document(indexName = "testgoods", type = "goods")
public class GoodsInfo implements Serializable {

    private Long id;
    private String name;
    private String description;

    public GoodsInfo() {

    }

    public GoodsInfo(Long id, String name, String description) {
        this.id = id;
        this.name = name;
        this.description = description;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }
}

```

###### 创建 GoodsInfoRepository.java 接口继承 ElasticsearchRepository  接口

```java
package com.ikang.springbootelasticsearch.repository;

import com.ikang.springbootelasticsearch.document.GoodsInfo;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
import org.springframework.stereotype.Component;

@Component
public interface GoodsRepository extends ElasticsearchRepository<GoodsInfo, Long> {

}

```

###### 创建 GoodsInfoController.java 

```java
package com.ikang.springbootelasticsearch.controller;

import com.ikang.springbootelasticsearch.document.GoodsInfo;
import com.ikang.springbootelasticsearch.repository.GoodsRepository;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightBuilder;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.elasticsearch.core.query.NativeSearchQuery;
import org.springframework.data.elasticsearch.core.query.NativeSearchQueryBuilder;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

@RestController
@RequestMapping("goods")
public class GoodsController {


    private final GoodsRepository goodsRepository;

    public GoodsController(GoodsRepository goodsRepository) {
        this.goodsRepository = goodsRepository;
    }

    @GetMapping("save")
    public String save() {
        GoodsInfo goodsInfo = new GoodsInfo(System.currentTimeMillis(),
                "商品" + System.currentTimeMillis(), "这是一个测试商品！");

        goodsRepository.save(goodsInfo);


        return "success";
    }

    @GetMapping("delete")
    public String delete(Long id) {
        goodsRepository.deleteById(id);

        return "success";
    }

    @GetMapping("update")
    public String update(Long id, String name, String description) {
        GoodsInfo goodsInfo = new GoodsInfo(id, name, description);
        goodsRepository.save(goodsInfo);

        return "success";
    }

    @GetMapping("getOne")
    public GoodsInfo getOne(Long id) {
        Optional<GoodsInfo> optional = goodsRepository.findById(id);

        return optional.isPresent() ? optional.get() : null;
    }

    @GetMapping("list")
    public List<GoodsInfo> list() {
        Iterable<GoodsInfo> all = goodsRepository.findAll();
        List<GoodsInfo> list = new ArrayList<>();
        all.forEach(list::add);
        return list;
    }

    // 每页查询数量
    private final Integer DEFAULT_PAGE_SIZE = 10;

    @GetMapping("getList")
        public Page<GoodsInfo> getList(Integer pageNum, String query) {

        Pageable page = PageRequest.of(pageNum, DEFAULT_PAGE_SIZE);

        NativeSearchQuery nativeSearchQuery = new NativeSearchQueryBuilder()
                .withPageable(page)
                .withQuery(QueryBuilders.matchAllQuery())
                .withFilter(QueryBuilders.matchQuery("name", query))
                .withFilter(QueryBuilders.matchQuery("description", query))
                .withHighlightFields(new HighlightBuilder.Field("name").preTags("<font color='red'>").postTags("</font>"),
                        new HighlightBuilder.Field("description").preTags("<font color='red'>").postTags("</font>"))
                .build();

        Page<GoodsInfo> goodsInfos = goodsRepository.search(nativeSearchQuery);

        return goodsInfos;
    }
}

```

#### 测试结果：

* 在浏览器中访问 http://localhost:8080/goods/save， http://localhost:8080/goods/update，http://localhost:8080/goods/list 将会看到所有数据已经插入，并且可以查询出来了：

```json
{
content: [
{
id: 1558662806732,
name: "商品1558662806732",
description: "这是一个测试商品！"
},
{
id: 1558662808245,
name: "商品1558662808245",
description: "这是一个测试商品！"
},
{
id: 1558662893476,
name: "商品1558662893476",
description: "这是一个测试商品！"
},
{
id: 1558662901290,
name: "商品1558662901290",
description: "这是一个测试商品！"
},
{
id: 1558662627656,
name: "商品1558662627656",
description: "这是一个测试商品！"
},
{
id: 1558662804196,
name: "商品1558662804196",
description: "这是一个测试商品！"
},
{
id: 1558662807792,
name: "商品1558662807792",
description: "这是一个测试商品！"
},
{
id: 1558662808433,
name: "商品1558662808433",
description: "这是一个测试商品！"
},
{
id: 1558662896876,
name: "商品1558662896876",
description: "这是一个测试商品！"
},
{
id: 1558662900326,
name: "商品1558662900326",
description: "这是一个测试商品！"
}
],
pageable: {
sort: {
unsorted: true,
sorted: false,
empty: true
},
offset: 10,
pageSize: 10,
pageNumber: 1,
paged: true,
unpaged: false
},
facets: [ ],
aggregations: null,
scrollId: null,
maxScore: 1,
totalElements: 26,
totalPages: 3,
size: 10,
number: 1,
sort: {
unsorted: true,
sorted: false,
empty: true
},
last: false,
numberOfElements: 10,
first: false,
empty: false
}
```

#### 遇到的问题：

* 1、启动报下面错误：

```bash
2019-05-24 11:26:57.753  WARN 9683 --- [           main] o.e.c.t.TransportClientNodesService      : node {#transport#-1}{qZfhmDjMRiKldY_XKULvNg}{127.0.0.1}{127.0.0.1:9300} not part of the cluster Cluster [elasticsearch], ignoring...
2019-05-24 11:26:58.201 ERROR 9683 --- [           main] .d.e.r.s.AbstractElasticsearchRepository : failed to load elasticsearch nodes : org.elasticsearch.client.transport.NoNodeAvailableException: None of the configured nodes are available: [{#transport#-1}{qZfhmDjMRiKldY_XKULvNg}{127.0.0.1}{127.0.0.1:9300}]
2019-05-24 11:26:58.350  INFO 9683 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-05-24 11:26:58.536  INFO 9683 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 2 endpoint(s) beneath base path '/actuator'
2019-05-24 11:26:58.588  INFO 9683 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-05-24 11:26:58.590  INFO 9683 --- [           main] c.i.s.SpringBootElasticsearchApplication : Started SpringBootElasticsearchApplication in 3.707 seconds (JVM running for 4.354)
2019-05-24 11:26:58.648  INFO 9683 --- [)-172.16.98.240] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2019-05-24 11:26:58.648  INFO 9683 --- [)-172.16.98.240] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2019-05-24 11:26:58.652  WARN 9683 --- [)-172.16.98.240] o.s.b.a.e.ElasticsearchHealthIndicator   : Elasticsearch health check failed

org.elasticsearch.client.transport.NoNodeAvailableException: None of the configured nodes are available: [{#transport#-1}{qZfhmDjMRiKldY_XKULvNg}{127.0.0.1}{127.0.0.1:9300}]
	at org.elasticsearch.client.transport.TransportClientNodesService.ensureNodesAreAvailable(TransportClientNodesService.java:349) ~[elasticsearch-6.4.3.jar:6.4.3]
	at org.elasticsearch.client.transport.TransportClientNodesService.execute(TransportClientNodesService.java:247) ~[elasticsearch-6.4.3.jar:6.4.3]
	at org.elasticsearch.client.transport.TransportProxyClient.execute(TransportProxyClient.java:60) ~[elasticsearch-6.4.3.jar:6.4.3]
	at org.elasticsearch.client.transport.TransportClient.doExecute(TransportClient.java:381) ~[elasticsearch-6.4.3.jar:6.4.3]
	at org.elasticsearch.client.support.AbstractClient.execute(AbstractClient.java:407) ~[elasticsearch-6.4.3.jar:6.4.3]
	at org.elasticsearch.client.support.AbstractClient.execute(AbstractClient.java:396) ~[elasticsearch-6.4.3.jar:6.4.3]
	at org.elasticsearch.client.support.AbstractClient$ClusterAdmin.execute(AbstractClient.java:708) ~[elasticsearch-6.4.3.jar:6.4.3]
	at org.elasticsearch.client.support.AbstractClient$ClusterAdmin.health(AbstractClient.java:730) ~[elasticsearch-6.4.3.jar:6.4.3]
	at org.springframework.boot.actuate.elasticsearch.ElasticsearchHealthIndicator.doHealthCheck(ElasticsearchHealthIndicator.java:79) ~[spring-boot-actuator-2.1.5.RELEASE.jar:2.1.5.RELEASE]
	at org.springframework.boot.actuate.health.AbstractHealthIndicator.health(AbstractHealthIndicator.java:84) ~[spring-boot-actuator-2.1.5.RELEASE.jar:2.1.5.RELEASE]
	at org.springframework.boot.actuate.health.CompositeHealthIndicator.health(CompositeHealthIndicator.java:98) [spring-boot-actuator-2.1.5.RELEASE.jar:2.1.5.RELEASE]
	at org.springframework.boot.actuate.health.HealthEndpoint.health(HealthEndpoint.java:50) [spring-boot-actuator-2.1.5.RELEASE.jar:2.1.5.RELEASE]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_211]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_211]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_211]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_211]
	at org.springframework.util.ReflectionUtils.invokeMethod(ReflectionUtils.java:282) [spring-core-5.1.7.RELEASE.jar:5.1.7.RELEASE]
	at org.springframework.boot.actuate.endpoint.invoke.reflect.ReflectiveOperationInvoker.invoke(ReflectiveOperationInvoker.java:76) [spring-boot-actuator-2.1.5.RELEASE.jar:2.1.5.RELEASE]
	at org.springframework.boot.actuate.endpoint.annotation.AbstractDiscoveredOperation.invoke(AbstractDiscoveredOperation.java:61) [spring-boot-actuator-2.1.5.RELEASE.jar:2.1.5.RELEASE]
	at org.springframework.boot.actuate.endpoint.jmx.EndpointMBean.invoke(EndpointMBean.java:126) [spring-boot-actuator-2.1.5.RELEASE.jar:2.1.5.RELEASE]
	at org.springframework.boot.actuate.endpoint.jmx.EndpointMBean.invoke(EndpointMBean.java:99) [spring-boot-actuator-2.1.5.RELEASE.jar:2.1.5.RELEASE]
	at com.sun.jmx.interceptor.DefaultMBeanServerInterceptor.invoke(DefaultMBeanServerInterceptor.java:819) [na:1.8.0_211]
	at com.sun.jmx.mbeanserver.JmxMBeanServer.invoke(JmxMBeanServer.java:801) [na:1.8.0_211]
	at javax.management.remote.rmi.RMIConnectionImpl.doOperation(RMIConnectionImpl.java:1468) [na:1.8.0_211]
	at javax.management.remote.rmi.RMIConnectionImpl.access$300(RMIConnectionImpl.java:76) [na:1.8.0_211]
	at javax.management.remote.rmi.RMIConnectionImpl$PrivilegedOperation.run(RMIConnectionImpl.java:1309) [na:1.8.0_211]
	at javax.management.remote.rmi.RMIConnectionImpl.doPrivilegedOperation(RMIConnectionImpl.java:1401) [na:1.8.0_211]
	at javax.management.remote.rmi.RMIConnectionImpl.invoke(RMIConnectionImpl.java:829) [na:1.8.0_211]
	at sun.reflect.GeneratedMethodAccessor54.invoke(Unknown Source) ~[na:na]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_211]

```

> 问题分析：
>
> 这是因为 springboot 中默认的 cluster-name 为 `elasticsearch`，我们用 docker 启动的 `elasticsearch` 默认的名称是 `docker-cluster`, 所以在  application.xml 中加入以下配置就可以解决问题：
>
> ```properties
> spring.data.elasticsearch.cluster-name=docker-cluster
> ```

* 2、在查询过程中会报一个无法反序列化的问题，这个问题的解决方法是在 GoodsInfo.java 中加入一个默认的无参构造方法，具体为什么，应该和 jackson 的序列化有关。