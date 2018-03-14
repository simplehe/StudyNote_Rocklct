## MongoDB数据库关系的表示和设计

在传统的SQL数据库中，关系被分为一个个表(table)，在表中，每个数据项以主键(primary key)标识，而一个表的主键又作为另一个表的外键（reference key)，在两个表之间引用。当遇上多对多关系的时候，还需要一个额外的关联表(reference table)，将多对多关系转化成两个一对多关系。

在MongoDB中有两种表示方法

#### 嵌套
一种是嵌套(embedded)，既是将一个文档包裹一个子文档；

每个MongoDB文档都由BSON文档组成，有类似JSON格式一样的数据类型，其中String、Int、Float称为基本类型(或常量)，而Hash和Array称之为复合类型。
 
所谓的嵌套，就是说文档中，利用复合类型，包裹一个多或多个其他类型的值，这些值称之为子文档。

这种方法很好理解，就是完全舍弃外键引用这种形式，直接在文档中保存需要的数据解决这个问题，比如用Array。
```
{'friends': [{'name': 'peter'}, {'name': 'john'}, {'name': 'marry'}], 'name': 'huangz'} 
```

如图，而传统的关系型数据库需要多次查表，得到外键以后要查找另一张表，不够直接。

#### 引用链接
比起嵌套，引用链接更接近传统意义上的(也就是，关系型数据库术语中的）“引用”，它是两个文档之间的一种关系。
 
引用链接通过DBRef对象建立，DBRef对象储存了如何找到目标文档的信息，就像现实世界中的门牌号码一样（也类似关系型数据库中的外键)。

如果在一个文档A中，有一个DBRef对象，而这个DBRef对象储存了关于如何找到文档B的信息，那么文档A就可以通过解释这个DBRef对象(称之为解引用)来获取文档B的数据。

<a>http://www.runoob.com/mongodb/mongodb-database-references.html</a>

DBRef的形式：
```
{ $ref : , $id : , $db :  }
```

- $ref：集合名称
- $id：引用的id
- $db:数据库名称，可选参数

看例子：
```
{
   "_id":ObjectId("53402597d852426020000002"),
   "address": {
   "$ref": "address_home",
   "$id": ObjectId("534009e4d852427820000002"),
   "$db": "w3cschoolcc"},
   "contact": "987654321",
   "dob": "01-01-1991",
   "name": "Tom Benzamin"
}
```

如图存储字段，address字段存放着别的表的引用，里面的$ref存放的就是另外一个表的名称，id是对应的外键，db是对应的数据库对象(选填)。这样就可以通过这个字段查到想要的信息

```
>var user = db.users.findOne({"name":"Tom Benzamin"})
>var dbRef = user.address
>db[dbRef.$ref].findOne({"_id":(dbRef.$id)})
```
以上实例返回了 address_home 集合中的地址数据：
```
{
   "_id" : ObjectId("534009e4d852427820000002"),
   "building" : "22 A, Indiana Apt",
   "pincode" : 123456,
   "city" : "Los Angeles",
   "state" : "California"
}
```