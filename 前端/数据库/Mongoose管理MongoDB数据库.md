## mongoose管理数据库
Node.js有针对MongoDB的数据库驱动：mongodb。你可以使用“npm install mongodb”来安装。

不过直接使用mongodb模块虽然强大而灵活，但有些繁琐，我就使用mongoose吧。

mongoose构建在mongodb之上，提供了**Schema、Model和Document**对象，用起来更为方便。

我们可以用Schema对象定义文档的结构（类似表结构），可以定义字段和类型、唯一性、索引和验证。Model对象表示集合中的所有文档。Document对象作为集合中的单个文档的表示。


注意mongodb不需要像关系型数据库一样设置主键，它自动会给每一个文档生成一个objectid的。

### 一些用法
Schema  ：  一种以文件形式存储的数据库模型骨架，不具备数据库的操作能力

Model   ：  由Schema发布生成的模型，具有抽象属性和行为的数据库操作对

Entity  ：  由Model创建的实体，他的操作也会影响数据库

相当于我们用Schema定义表的格式，Model用来创建具体的表。

``` javascript
    var PersonSchema = new mongoose.Schema({
      name:String   //定义一个属性name，类型为String
    });
```

这样相当于创建了一个表模式

``` javascript
    var PersonModel = db.model('Person',PersonSchema);
    //如果该Model已经发布，则可以直接通过名字索引到，如下：
    //var PersonModel = db.model('Person');
    //如果没有发布，上一段代码将会异常
```

相当于创建具体的表，表名为person

#### 给model创建自带的方法
我们给Schema创建方法，生成的model将会继承到这些方法。

``` javascript
    //为Schema模型追加speak方法
    PersonSchema.methos.speak = function(){
      console.log('我的名字叫'+this.name);
    }
    var PersonModel = db.model('Person',PersonSchema);
    var person = new PersonModel({name:'Krouky'});
    person.speak();//我的名字叫Krouky
```

根据model创建的实例为具体的document，document对象调用save方法就是保存到数据库中。

``` javascript
person.save(function (err, fluffy) {
  if (err) return console.error(err);
});
```
注意model保存到database中的话，collection名(即是表名) 会自动转换成为复数。比如user，用mongoose保存之后存到数据库里会是users

#### 查询
model对象自带一些方法用于查询。

```
var Person = mongoose.model('Person', yourSchema);
// find each person with a last name matching 'Ghost', selecting the `name` and `occupation` fields
Person.findOne({ 'name.last': 'Ghost' }, 'name occupation', function (err, person) {
  if (err) return handleError(err);
  console.log('%s %s is a %s.', person.name.first, person.name.last, person.occupation) // Space Ghost is a talk show host.
})
```

有find方法，findByOne方法等，具体见文档。

需要传递进一个回调函数来进行数据的调用。

注意，也可以不传入回调函数先，如果不传入回调函数，它就只是一个query对象，不会执行，需要调用exec()来执行，如下。
``` javascript
Person
.find({ occupation: /host/ })
.where('name.last').equals('Ghost')
.where('age').gt(17).lt(66)
.where('likes').in(['vaporizing', 'talking'])
.limit(10)
.sort('-occupation')
.select('name occupation')
.exec(callback);
```