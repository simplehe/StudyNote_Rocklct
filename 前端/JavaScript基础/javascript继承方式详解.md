## JavaScript继承方式详解

js里常用的如下两种继承方式：

 - 原型链继承（对象间的继承）
 - 类式继承（构造函数间的继承）

所以，要想实现继承，可以用js的原型prototype机制或者用apply和call方法去实现

### 原型式继承与类式继承
类式继承是在子类型构造函数的内部调用超类型的构造函数。
严格的类式继承并不是很常见，一般都是组合着用：

``` javascript
function Super(){
    this.colors=["red","blue"];
}
 
function Sub(){
    Super.call(this);
}
```

原型式继承是借助已有的对象创建新的对象，将子类的原型指向父类，就相当于加入了父类这条原型链

### 原型链继承
为了让子类继承父类的属性（也包括方法），首先需要定义一个构造函数。然后，将父类的新实例赋值给构造函数的原型。代码如下：

``` html
<script>
    function Parent(){
        this.name = 'mike';
    }

    function Child(){
        this.age = 12;
    }
    Child.prototype = new Parent();//Child继承Parent，通过原型，形成链条

    var test = new Child();
    alert(test.age);
    alert(test.name);//得到被继承的属性
    //继续原型链继承
    function Brother(){   //brother构造
        this.weight = 60;
    }
    Brother.prototype = new Child();//继续原型链继承
    var brother = new Brother();
    alert(brother.name);//继承了Parent和Child,弹出mike
    alert(brother.age);//弹出12
</script>
```

以上原型链继承还缺少一环，那就是Object，所有的构造函数都继承自Object。而继承Object是自动完成的，并不需要我们自己手动继承，那么他们的从属关系是怎样的呢？