## 关系型数据库(RDBMS)遵循ACID规则

**1、A (Atomicity) 原子性**

**2、C (Consistency) 一致性**

**3、I (Isolation) 独立性(隔离性)**

**4、D (Durability) 持久性**

## 非关系型数据库(NoSQL)

## CAP定理

1. **一致性(Consistency)** (所有节点在同一时间具有相同的数据)
2. **可用性(Availability)**(保证每个请求不管成功或者失败都有响应)
3. **分割容忍(Partition tolerance) **(系统中任意信息的丢失或失败不会影响系统的继续运作)

##### CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个



## MongoDB是文档存储，默认端口：27017

#### 特点：文档存储一般用类似json的格式存储，存储的内容是文档型的。这样也就有有机会对某些字段建立索引，实现关系数据库的某些功能。

**MongoDB**是由C++语言编写的，是一个基于分布式文件存储的开源数据库系统。



## Mac 安装MongoDB

教程地址：https://www.cnblogs.com/weixuqin/p/7258000.html

多实例启动教程：https://cloud.tencent.com/info/996c48bd5ab5a5b91a754213beee24b4.html



## MongoDB的基本使用

1. 启动MongoDB服务：

```
brew services start mongodb@3.4#重命名为smo
```

2. 关闭MongoDB服务：

```
brew services stop mongodb@3.4 #重命名为emo
```

3. 进入MongoDB图形化界面：

```
mongo
```



#### 创建数据库(MongoDB 中默认的数据库为 test，如果你没有创建新的数据库，集合将存放在 test 数据库中)

```python
use DATABASE_NAME
```

#### 查看当前使用的数据库

```python
db
```

#### 查看所有的数据库(新建的数据库如果没有插入数据，则不显示)

```python
show dbs
```

#### 删除当前数据库(如果没有选择数据库，则删除test)

```python
db.dropDatabase()
```



#### 向数据库中插入一条数据(MongoDB会自动创建集合)

```python
db.COLLECTION_NAME.insert(document)
db.mongo01.insert({'name':'老王'})
```

#### 显示当前数据库下的所有文档

```python
show tables
```

#### 删除文档

```python
db.TABLE_NAME.drop()
```



#### #### 创建集合

```python
db.createCollection(name,options)
```

参数说明：

- name: 要创建的集合名称
- options: 可选参数, 指定有关内存大小及索引的选项

options 可以是如下参数：

| 字段        | 类型 | 描述                                                         |
| ----------- | ---- | ------------------------------------------------------------ |
| capped      | 布尔 | （可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。 **当该值为 true 时，必须指定 size 参数。** |
| autoIndexId | 布尔 | （可选）如为 true，自动在 _id 字段创建索引。默认为 false。   |
| size        | 数值 | （可选）为固定集合指定一个最大值（以字节计）。 **如果 capped 为 true，也需要指定该字段。** |
| max         | 数值 | （可选）指定固定集合中包含文档的最大数量。                   |

在插入文档时，MongoDB 首先检查**固定集合**的 size 字段，然后检查 max 字段。

```python
use mongo01
db.createCollection('collecton01')

db.createCollection('collection02',{capped:true,
                                    autoIndexId:true,
                                    size:23456456,
                                    max:10000})
```

#### 显示所有的集合

```python
show collections
```

#### 删除集合

```python
db.collection_name.drop()
```



#### 向集合中插入文档

```python
document={
    title: 'MongoDB 教程', 
    description: 'MongoDB 是一个 Nosql 数据库',
    by: '菜鸟教程',
    url: 'http://www.runoob.com',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
}


db.col.insert(document)
```



#### 插入单条文档

```python
db.colection_name.insertOne({'a':3})
```

#### 插入多条文档数据

```python
db.COLLECTION_NAME.insertMany(
      [
         {
           _id: 1,
           name: "sue",
           age: 19,
           type: 1,
           status: "P",
           favorites: { artist: "Picasso", food: "pizza" },
           finished: [ 17, 3 ],
           badges: [ "blue", "black" ],
           points: [
              { points: 85, bonus: 20 },
              { points: 85, bonus: 10 }
           ]
         },
         {
           _id: 2,
           name: "bob",
           age: 42,
           type: 1,
           status: "A",
           favorites: { artist: "Miro", food: "meringue" },
           finished: [ 11, 25 ],
           badges: [ "green" ],
           points: [
              { points: 85, bonus: 20 },
              { points: 64, bonus: 12 }
           ]
         },
         {
           _id: 3,
           name: "ahn",
           age: 22,
           type: 2,
           status: "A",
           favorites: { artist: "Cassatt", food: "cake" },
           finished: [ 6 ],
           badges: [ "blue", "red" ],
           points: [
              { points: 81, bonus: 8 },
              { points: 55, bonus: 20 }
           ]
         },
         {
           _id: 4,
           name: "xi",
           age: 34,
           type: 2,
           status: "D",
           favorites: { artist: "Chagall", food: "chocolate" },
           finished: [ 5, 11 ],
           badges: [ "red", "black" ],
           points: [
              { points: 53, bonus: 15 },
              { points: 51, bonus: 15 }
           ]
         },
         {
           _id: 5,
           name: "xyz",
           age: 23,
           type: 2,
           status: "D",
           favorites: { artist: "Noguchi", food: "nougat" },
           finished: [ 14, 6 ],
           badges: [ "orange" ],
           points: [
              { points: 71, bonus: 20 }
           ]
         },
         {
           _id: 6,
           name: "abc",
           age: 43,
           type: 1,
           status: "A",
           favorites: { food: "pizza", artist: "Picasso" },
           finished: [ 18, 12 ],
           badges: [ "black", "blue" ],
           points: [
              { points: 78, bonus: 8 },
              { points: 57, bonus: 7 }
           ]
         }
      ]
    )
```



#### 更新文档 update()方法

```python
db.COLLECTION.update(
<query>, #查询条件,类似于sql update查询内where后面的
<update>,#update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
    {
        upsert:<boolean>,#可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入
        multi:<boolean>,#可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新
        writeConcern:<docuemnt>#可选，抛出异常的级别。
    }
)
```

更新多条记录：

```python
db.test_collection.updateMany({"age":{$gt:"10"}},{$set:{"status":"xyz"}})
#将age大于10的文档中的status参数的数据改为xyz
```

更新单条记录：

```python
db.test_collection.updateOne({"name":"abc"},{$set:{"age":"28"}})
```



#### 更新文档 save()方法

```python
db.collection.save(
<docuement>, #文档数据,文档重写后保存
{
writeConcern:<document> #可选，抛出异常的级别
}
)
```



#### 删除文档

```python
db.collection_name.remove(
<query>,  #可选，删除的文档的条件
{
     justOne: <boolean>, #可选，如果设为true或者1，则只删除一个文档
     writeConcern: <document> #可选，抛出异常的级别
   }
)
```

删除所有数据

```python
db.COLLECTION_NAME.deleteMany({})
```

删除status等于A的全部文档

```python
db.COLLECTION_NAME.deleteMany({status:'A'})
```

删除status等于D的一个文档

```python
db.COLLECTION.deleteOne({status:'D'})
```



#### 删除文档后，查看索引，索引依然存在

```python
db.collection_name.getIndexes()
```

删除存在的索引

```python
db.collection_name.dropIndexes()
```



#### 查看文档数据

```python
db.COLLECTION_NAME.find(query,projection)
# query ：可选，使用查询操作符指定查询条件
# projection ：可选，使用投影操作符指定返回的键

#格式化显示
db.COLLECTION_NAME.find().pretty()

#根据条件查询文档数据
db.COLLECTION_NAME.find(status:'A').count()

db.col.find({"by":"菜鸟教程"}).pretty()	where by = '菜鸟教程' #等于
db.col.find({"likes":{$lt:50}}).pretty() #小于 less than
db.col.find({"likes":{$lte:50}}).pretty()# 小于等于less than equal
db.col.find({"likes":{$gt:50}}).pretty()# 大于 greater than
db.col.find({"likes":{$gte:50}}).pretty()# 大于等于 greater than equal
db.col.find({"likes":{$ne:50}}).pretty()#不等于 not equal
```

#### and条件(逗号分隔多个查询条件)

```python
db.COLLECTION_NAME.find({key1:value1,key2:value2}).pretty()
```

#### or条件

```python
db.COLLECTION_NAME.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()

#例子：
db.col.find({$or:[{"by":"菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()
```

#### and和or联合使用

```python
db.col.find({"likes": {$gt:50}, $or: [{"by": "菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()
```

```python
db.collection.find(query, {title: 1, by: 1}) # inclusion模式 指定返回的键，不返回其他键
db.collection.find(query, {title: 0, by: 0}) # exclusion模式 指定不返回的键,返回其他键
```

若不想指定查询条件参数 **query** 可以 用 {} 代替，但是需要指定 **projection** 参数：

```python
querydb.collection.find({}, {title: 1})
```



#### $type操作符

| **类型**                | **数字** | **备注**         |
| ----------------------- | -------- | ---------------- |
| Double                  | 1        |                  |
| String                  | 2        |                  |
| Object                  | 3        |                  |
| Array                   | 4        |                  |
| Binary data             | 5        |                  |
| Undefined               | 6        | 已废弃。         |
| Object id               | 7        |                  |
| Boolean                 | 8        |                  |
| Date                    | 9        |                  |
| Null                    | 10       |                  |
| Regular Expression      | 11       |                  |
| JavaScript              | 13       |                  |
| Symbol                  | 14       |                  |
| JavaScript (with scope) | 15       |                  |
| 32-bit integer          | 16       |                  |
| Timestamp               | 17       |                  |
| 64-bit integer          | 18       |                  |
| Min key                 | 255      | Query with `-1`. |
| Max key                 | 127      |                  |

```python
db.col.find({'name':{$type:2}}) #将文档中name设置为String的数据
```

#### limit()方法

```python
db.COLLECTION_NAME.find().limit(NUMBER)
```

#### skip()方法(跳过指定数量的数据，默认为0)

```python
db.COLLECTION_NAME.find().skip(NUMBER)

db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)                         
```

#### sort()方法,1代表升序，-1代表降序

```python
db.COLLECTION_NAME.find().sort({KEY:1})

db.col.find({},{'name':1,_id:0}).sort({'age':-1})
```

#### createIndex()方法,1为升序，-1为降序

```python
db.collection_name.createIndex(keys,options)

db.col.createIndex({'title':1})
db.col.createIndex({'title':1,'description':-1})#关系型数据库中称为复合索引
```



#### 聚合函数

**aggregate()方法**

```python
db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)
```

| 表达式    | 描述                                           | 实例（其实中括号可以不要）                                   |
| --------- | ---------------------------------------------- | ------------------------------------------------------------ |
| $sum      | 计算总和。                                     | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : "$likes"}}}]) |
| $avg      | 计算平均值                                     | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$avg : "$likes"}}}]) |
| $min      | 获取集合中所有文档对应值得最小值。             | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$min : "$likes"}}}]) |
| $max      | 获取集合中所有文档对应值得最大值。             | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$max : "$likes"}}}]) |
| $push     | 在结果文档中插入值到一个数组中。               | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$push: "$url"}}}]) |
| $addToSet | 在结果文档中插入值到一个数组中，但不创建副本。 | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$addToSet : "$url"}}}]) |
| $first    | 根据资源文档的排序获取第一个文档数据。         | db.mycol.aggregate([{$group : {_id : "$by_user", first_url : {$first : "$url"}}}]) |
| $last     | 根据资源文档的排序获取最后一个文档数据         | db.mycol.aggregate([{$group : {_id : "$by_user", last_url : {$last : "$url"}}}]) |

**管道操作符**

- $project：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
- $match：用于过滤数据，只输出符合条件的文档。$match使用MongoDB的标准查询操作。
- $limit：用来限制MongoDB聚合管道返回的文档数。
- $skip：在聚合管道中跳过指定数量的文档，并返回余下的文档。
- $unwind：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。(key:[value1,value2,…],将value拆开，其他元素不变，组成一个新的文档)
- $group：将集合中的文档分组，可用于统计结果。
- $sort：将输入文档排序后输出。
- $geoNear：输出接近某一地理位置的有序文档。

**project实例**

```python
db.col.aggregate({$project:{
... name:1, #想显示谁就把谁设置为1
... age:1,
... _id:0}}) #_id是默认显示的，如果不想它显示，则设置_id:0
```

**match实例**

```python
db.col.aggregate({$match:{age:{$gt:28,$lt:40}}})

#和group聚合使用
db.col.aggregate([{$match:{age:{$gt:28,$lt:40}}}, {$group:{_id:'$job',count:{$sum:'$age'}}}])
```

**skip**

```python
db.col.aggregate({$skip:4})
```

**sort**

```python
db.col.aggregate({$sort:{'age':-1}}) #以age字段的值倒叙排列
```

## mongodb 数据库引用

1. DBRefs

   ```python
   {$ref:,$id:,$db:} #集合名称，引用的id，数据库名称(可选)
   ```

   

#### MongoDB常用方法见地址：

https://blog.csdn.net/eagle89/article/details/80609343



#### 创建索引

```python
db.collection_name.ensureIndex({field:index_name})#对某个字段创建一个索引，并设置一个索引名
```

#### 查看索引

```python
db.collection_name.getIndexes()
```

#### 删除索引

```python
db.colletion_name.dropIndex('index_name')
```



#### 验证索引是否使用

```python
db.collection_name.find({field:value}).explain()

db.users.ensureIndex({"address.city":1,"address.state":1,"address.pincode":1})
#1代表正序，address字段下面有city,state,pincode三个子字段
#在观察cursor是否是basiccursor还是BtreeCursor
```

## MongoDB Map Reduce 计算模型

将大批量的工作（数据）分解（MAP）执行，然后再将结果合并成最终结果（REDUCE）









