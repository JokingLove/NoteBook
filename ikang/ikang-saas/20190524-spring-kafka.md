## Kafka (a distributed streaming platform)

#### Kafka 的三大功能：

- 发布 & 订阅

  类似于一个消息系统，读写流式的数据。									

- 处理

  编写可扩展的流处理应用程序，用于实时时间相应的场景。

- 存储

  安全的将流式的数据存储在一个分布式、有副本备份、容错的集群。

![20190524152850-image.png](http://ps01v423x.bkt.clouddn.com/2019/05/24/kafka_diagram.png?imageView2/0/w/400/h/500)

#### Kafka 适合的场景：

​	Kafka 主要应用于两大类别的应用

* 1、构造实时流数据管道，它可以在系统或应用之间可靠的获取数据。(相当于 message queue)
* 2、构建实时流式应用程序，对这些流式数据进行转换或者影响。(就是流处理，通过 kafka strem topic 和 topic 之间内部进行变化)

#### 一些概念

- Kafka 作为一个集群，运行在一台或者多台服务器上。
- Kafka 通过 topic 对存储的流数据进行分类。
- 每天记录包含一个 key ，一个 value 和一个 timestamp (时间戳)。

#### Kafka 的四个核心 API

* The Produce API 允许一个应用程序发布一串流式数据到一个或者多个 Kafka topic。

* The Consumer API 允许一个应用程序订阅一个或者多个 topic，并对发布给他们的流式数据进行处理。

* The Stream API 允许一个应用程序作为一个流处理器，消费一个或者多个 topic 产生的输入流，然后生产一个输出流到一个或者多个 topic 中去，在输入输出流中进行有效的转换。

* The Connector API 允许构建并运行可重用的生产者或消费者，将 Kafka topic 连接到已存在的应用程序或者数据系统。比如，连接到一个关系型数据库，捕捉表 (table) 的所有变更内容。

  ![kafka-apis](http://ps01v423x.bkt.clouddn.com/2019/05/24/kafka-apis.png?imageView2/0/w/500/h/500)

