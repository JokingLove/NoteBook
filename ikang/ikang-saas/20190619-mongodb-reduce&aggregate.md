## Mongodb 中两个操作 Reduce & Aggregate



### Map Reduce

Map-Reduce 是一种计算模型，简单的说就是将大批量的工作(数据) 分解 (MAP) 执行，然后再将结果合并成最终的结果 (REDUCE)。

MongoDB 提供的 Map-Reduce 非常灵活，对于大规模数据分析也相当实用。

---

#### MapReduce 命令

以下是 MapReduce 的基本语法：

```mongodb
> db.collection.mapReduce(
	funciton() {emit(key, value);}, // map 函数
	funciton(key, value) {return reduceFunction}, // reduce 函数
	{
		out: collection,
		query: document,
		sort: document,
		limit: number
	}
)
```

实用 MapReduce 要实现两个函数 Map 函数和 Reduce 函数， Map 函数调用 emit(key, value)，遍历 collection 中所有的记录，将 key 与 value 传递给 Reduce 函数进行处理。

Map 函数必须调用 emit(key, value) 返回键值对。

参数说明：

* map: 映射函数 (生成键值对序列，作为 reduce 函数的参数)。
* reduce： 统计函数，reduce 函数的任务就是将 key-value ，也就是把 values 数组变成一个单一的值 value。
* out： 统计函数存放集合 (不指定则使用临时集合，在客户端断开后自动删除)。
* query：一个筛选条件，只有满足条件的文档才会调用 map 函数。(query、limit、sort 可以随意组合)
* sort：和 limit 结合的 sort 排序菜蔬 (也是发往 map 函数前给文档排序)，可以优化分组机制。
* limit：发往 map 函数的文档数量的上限 (要是没有 limit， 单独使用  sort 的用处不大)。

---

#### 使用 MapReduce

考虑一下文档结构存储用户的文章，文档存储了用户的 user_name 和文章的 status 字段：

```mongodb
>db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "mark",
   "status":"active"
})
WriteResult({ "nInserted" : 1 })
>db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "mark",
   "status":"active"
})
WriteResult({ "nInserted" : 1 })
>db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "mark",
   "status":"active"
})
WriteResult({ "nInserted" : 1 })
>db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "mark",
   "status":"active"
})
WriteResult({ "nInserted" : 1 })
>db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "mark",
   "status":"disabled"
})
WriteResult({ "nInserted" : 1 })
>db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "runoob",
   "status":"disabled"
})
WriteResult({ "nInserted" : 1 })
>db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "runoob",
   "status":"disabled"
})
WriteResult({ "nInserted" : 1 })
>db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "runoob",
   "status":"active"
})
WriteResult({ "nInserted" : 1 })
```

现在，我们将在 posts 集合中使用  mapReduce 函数来选取已发布的文章 (status: "active"), 并通过 user_name 分组，计算每个用户的文章数：

```mongo
> db.posts.mapReduce(
	function() { emit(this.user_name, 1);},
	function(key, value) {return Array.sum(values)},
	{
		query: {status: "active"},
		out: "post_total"
	}
)
```

以上 mapReduce 输出结果为：

```mongo
{
        "result" : "post_total",
        "timeMillis" : 23,
        "counts" : {
                "input" : 5,
                "emit" : 5,
                "reduce" : 1,
                "output" : 2
        },
        "ok" : 1
}
```

结果表明，共有 4 个符合查询条件 (status: "active") 的文档，在 map 函数中生成了 4 个键值对文档，最后使用 reduce 函数将相同的键值分为两组。

具体参数说明：

* result：存储结果的 collection 的名字，这是个临时集合， MapReduce 的连接关闭后就自动被删除了。
* timeMillis：执行花费的时间，毫秒为单位
* input：满足条件被发送到  map 函数的文档个数。
* emit：在 map 函数中 emit 被调动的次数，也就是所有集合找那个的数据总量。
* output：结果结合中的文档个数 <font color=red>(count 对调试非常有帮助）</font>
* ok： 是否成功，成功为 1。
* err：如果失败，这里可以有失败原因，不过从经验来看，原因比较模糊，作用不大。

使用  find 操作符来查看 mapReduce 的查询结果：

```mongo
>db.posts.mapReduce( 
   function() { emit(this.user_name,1); }, 
   function(key, values) {return Array.sum(values)}, 
      {  
         query:{status:"active"},  
         out:"post_total" 
      }
).find()
```

以上查询显示如下结果，两个用户 tom 和 mark 有两个发布的文章：

```mongo
{ "_id" : "mark", "value" : 4 }
{ "_id" : "runoob", "value" : 1 }
```

用类似的方式，MapReduce可以被用来构建大型复杂的聚合查询。

Map函数和Reduce函数可以使用 JavaScript 来实现，使得MapReduce的使用非常灵活和强大。



### Aggregate 

MongoDB 中聚合 (aggregate) 主要用于处理数据 (诸如统计平均值，求和等)，并返回计算后的数据结果。有点类似  sql 语句中的 count(*)。

---

#### aggregate() 方法

MongoDB 中聚合的方法使用 aggregate()。

##### 语法

aggregate() 方法的基本语法格式如下所示：

```mongo
>db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)
```

##### 实例

集合中的数据如下：

```json
{     
	_id: ObjectId(7df78ad8902c)    
	title: 'MongoDB Overview',      
	description: 'MongoDB is no sql database',   
	by_user: 'w3cschool.cc',   
	url: 'http://www.w3cschool.cc',  
	tags: ['mongodb', 'database', 'NoSQL'],  
	likes: 100  
}, 
{  
	_id: ObjectId(7df78ad8902d)     
	title: 'NoSQL Overview',    
	description: 'No sql database is very fast',    
	by_user: 'w3cschool.cc',   
	url: 'http://www.w3cschool.cc',   
	tags: ['mongodb', 'database', 'NoSQL'],   
	likes: 10  
},  
{ 
	_id: ObjectId(7df78ad8902e)    
	title: 'Neo4j Overview',  
	description: 'Neo4j is no sql database',   
	by_user: 'Neo4j',   
	url: 'http://www.neo4j.com',  
	tags: ['neo4j', 'database', 'NoSQL'],  
	likes: 750  
}
```

现在我们通过以上集合计算每个作者所写的文章数，使用 aggregate() 计算结果如下：

```bash
> db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}]) 
{    
	"result" : [      
		{          
			"_id" : "w3cschool.cc",       
			"num_tutorial" : 2     
		},       
		{         
			"_id" : "Neo4j",       
			"num_tutorial" : 1     
		}     
	],    
	"ok" : 1 
}  
> 
```

以上实例类似 sql 语句： select by_user, count(*) from mycol group by by_user;

在上面的例子中，我们通过字段 by_user 字段对数据进行分组，并计算 by_user 字段相同值得综合。

下标展示了一些集合的表达式：

| 表达式    | 描述                                           | 实例                                                         |
| :-------- | :--------------------------------------------- | :----------------------------------------------------------- |
| $sum      | 计算总和。                                     | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : "$likes"}}}]) |
| $avg      | 计算平均值                                     | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$avg : "$likes"}}}]) |
| $min      | 获取集合中所有文档对应值得最小值。             | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$min : "$likes"}}}]) |
| $max      | 获取集合中所有文档对应值得最大值。             | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$max : "$likes"}}}]) |
| $push     | 在结果文档中插入值到一个数组中。               | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$push: "$url"}}}]) |
| $addToSet | 在结果文档中插入值到一个数组中，但不创建副本。 | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$addToSet : "$url"}}}]) |
| $first    | 根据资源文档的排序获取第一个文档数据。         | db.mycol.aggregate([{$group : {_id : "$by_user", first_url : {$first : "$url"}}}]) |
| $last     | 根据资源文档的排序获取最后一个文档数据         | db.mycol.aggregate([{$group : {_id : "$by_user", last_url : {$last : "$url"}}}]) |

---

#### 管道的概念

管道在 Unix 和 Linux 中一般用于将当前命令的输出的结果作为下一个命令的参数。

MongoDB 的聚合管道将 MongoDB 文档在一个管道处理完毕后将结果传递给下一个管道处理。管道操作是可以重复的。

表达式：处理输入文档并输出。表达式是无状态的，只能用于计算当前聚合管道的文档，不能处理其他的文档。这里我们介绍一下聚合框架中常用的几个操作：

* $project: 修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
* $match: 用于过滤数据，只输出符合条件的文档。`\$match` 使用 MongoDB 的标准查询操作。
* $limit: 用来限制 MongoDB 聚合管道返回的文档数。
* $skip：在聚合管道中跳过指定数量的文档，并返回余下的文档。
* $unwind: 将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
* $group: 将集合中的文档分组，可用于统计结果。
* $sort: 将输入文档排序后输出。
* $geoNear: 数据接近某一地理位置的有序文档。

##### 管道操作符实例

1. $project实例

```mongo
db.article.aggregate(     
	{
		$project : {         
			title : 1 ,      
			author : 1 ,   
		}
	}
); 
```

这样的话结果中就只还有_id,tilte和author三个字段了，默认情况下_id字段是被包含的，如果要想不包含_id话可以这样:

```
db.article.aggregate(    
	{ 
		$project : { 
			_id : 0 ,       
			title : 1 ,   
			author : 1     
		}
	}
); 
```

2. $match实例

```
db.articles.aggregate( [   
	{ $match : { score : { $gt : 70, $lte : 90 } } },  
	{ $group: { _id: null, count: { $sum: 1 } } }                  
]);  
```

3. $skip实例

```
db.article.aggregate(      
	{ $skip : 5 }
);   
```

经过$skip管道操作符处理后，前五个文档被"过滤"掉。