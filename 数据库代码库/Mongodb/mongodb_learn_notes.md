## Mongodb Learning Notes<br>
记录自己学习mongodb数据库中的知识点。mongodb数据库的教程除了查看官方文档，还可以在[易百教程-mongodb教程](http://www.yiibai.com/mongodb/)上查看。<br>

* Mongdb数据库的安装<br>
  Mongdodb数据库的安装挺简单的，总的来说就以下几步：<br>
  (1)到mongodb[官网](http://www.mongodb.org/downloads)下载安装包。<br>
  (2)解压安装包，创建mongodb一些数据目录，日志目录等。<br>
  (3)根据(2)创建的文件夹目录，配置mongodb数据库的配置文件mongodb.conf。更多详情参考：[Ubuntu下离线安装MongoDB教程](http://blog.csdn.net/u010858605/article/details/50957610)<br> windows下的安装可以参考:[MongoDB数据库简介及安装](http://www.cnblogs.com/libingql/archive/2011/06/13/2079438.html)<br>
* Mongodb针对数据库的基本命令：<br>
  (1) 开启mongodb数据库服务：（Linux下）<br>
```shell
# 这样启动服务要先把mongodb安装成服务
# service mongodb start 
或者
# 通过mongod命令结合配置文件启动服务
# mongod --config /etc/mongodb.conf
```
   (2) 进入mongodb数据库<br>
```shell
## Linux
# mongo

## windows
>mongo.exe
```
   (3)其他一些数据库级别的命令:<br>
```shell
　　show dbs: 查看所有数据库

    db : 查看当前所选的数据库    

　　help: 查看帮助文档

　　db.help(): 在数据库级别查看帮助信息

　　db.users.help(): 在集合级别查看帮助信息

　　db.dropDatabase(): 删除数据库

    use test : 选择一个数据库(可以直接随便选择一个数据库，若是后面在该数据库中没有写入集合数据，那么不会保存)
```
* Mongodb针对集合的操作:(类似关系型中的表)<br>
  在NOSQL类的数据库中，mongodb中的集合就等价于关系型中的表table的概念。这个保存数据的最小容器。<br>
```shell
   > db.createCollection("xxx")  ## 创建名为xxx的集合容器
   > show collections            ## 查询当前数据库中所有的集合
   > db.xxx.drop()               ## 删除当前数据库中名为xxx的集合
```
* Mongodb数据库集合文档数据的操作:<br>
  当选择好数据库，创建好集合后，就可以在集合中对数据进行增删改查了<br>
```shell
   > use test   ##选择或者创建test数据库
   > db.createCollection("users") ##创建users集合
   
   ## ----- 插入文档数据 ------ ##
   > db.users.insert([
    {  name:"jamx", eamil:"jamx@qq.com"},
    {  name:"tom", email:"tom@qq.com"}
    ])
   >  db.users.save([                    ## 也可以用save插入，但与insert有差别
    {  name:"jamx", eamil:"jamx@qq.com"},
    {  name:"tom", email:"tom@qq.com"}
    ])

   ## ----- 更新指定文档数据 ------ ##
   ## 适用$set: 命令来更新指定文档中指定字段，下面意思：更新名字为tom的邮箱地址
   ## 为tomupdate@qq.com
   > db.users.update({"name":"tom"},{$set:{"email":"tomupdate@qq.com"}})
   
   ## ------删除指定文档数据------##
   ## 通过remove函数中，指定查询参数进行删除，删除名字为tom的文档数据
   > db.users.remove({"name":"tom"})
   
   ## ------查询文档数据 ---------##
   > db.users.find().pretty()  ## 查询users集合中所有文档数据，加上pretty函数用于输出美化格式
   > db.users.find({"name":"tom","email":"tom@qq.com"}) ## AND查询，同时满足名字为tom,邮箱为tom@qq.com的文档数据
   > db.users.find({                                    ## OR查询使用$or:命令
      $or:[
           "name":"tom",
           "email":"jamx@qq.com"
       ]
      })
    > db.users.find({age:{$gt:20}})     ## age大于20的文档。xxx:{$gt/$lt:/$lte/$gte : xx} 
   
    
   
```
* Mongodb数据类型:<br>
  Mongodb数据库中的数据类型有以下几种:<br>
```shell
String : 这是最常用的数据类型来存储数据。在MongoDB中的字符串必须是有效的UTF-8。

Integer : 这种类型是用来存储一个数值。整数可以是32位或64位，这取决于您的服务器。

Boolean : 此类型用于存储一个布尔值 (true/ false) 。

Double : 这种类型是用来存储浮点值。

Min/ Max keys : 这种类型被用来对BSON元素的最低和最高值比较。

Arrays : 使用此类型的数组或列表或多个值存储到一个键。

Timestamp : 时间戳。这可以方便记录时的文件已被修改或添加。

Object : 此数据类型用于嵌入式的文件。

Null : 这种类型是用来存储一个Null值。

Symbol : 此数据类型用于字符串相同，但它通常是保留给特定符号类型的语言使用。

Date : 此数据类型用于存储当前日期或时间的UNIX时间格式。可以指定自己的日期和时间，日期和年，月，日到创建对象。

Object ID : 此数据类型用于存储文档的ID。

Binary data : 此数据类型用于存储二进制数据。

Code : 此数据类型用于存储到文档中的JavaScript代码。

Regular expression : 此数据类型用于存储正则表达式
```
若是想查找文档中name字段是字符串的文档记录:可以使用**$type**<br>
```shell
> db.users.find({"name":{$type:2}})
```
若是想将users集合中age文档数据key值的字符串类型改为int整型类型:<br>
```shell
> db.users.find({'age':{$type 2}}).forEach(function(x){x.age = new NumberInt(x.age); db.users.save(x);})
```
* Mongodb数据库的分页与排序<br>
  若是在mongodb中想要查询指定数量的文档数据，可以使用**limit**语法；若是想要将集合中文档数据进行排序使用**sort**语法。<br>
```shell
> db.users.find().limit(2)   ## 读取最前面的两条记录

## 排序使用sort命令，在参数中传入key值，value值为1：升序，-1：降序
> db.users.find().sort({"name":1})
```
* Mongodb数据库的索引<br>
 索引的创建：`db.collections.ensureIndex({key:1|-1}...)`<br>
 索引的删除：`db.collections.dropIndex({key:1|-1}...)`<br>
```shell
## 分别在name,email字段上创建索引，升序
> db.users.ensureIndex({"name":1},{"email":1})

## 创建name,email的联合索引
> db.uses.ensureIndex({"name":1,"email":1})

## 查询集合中的索引
> db.users.getIndexes()

## 删除指定集合中的索引
> db.users.dropIndex({"name":1})

```
* Mongodb数据导出<br>
  这里使用mongoexport命令，可以将指定条件的文档数据导入到指定目录下的文件中。<br>
```shell
## 导出test数据库中users集合中年龄大于30的文档数据，保存到/tmp/users.json文件中
# mongoexport -d test -c users -q '{age : {$gt:30}}' -o /tmp/users.json
```