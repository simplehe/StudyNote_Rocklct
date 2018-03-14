## sequelize
sequelize是一个ORM模型，用来帮助你通过面向对象代码操作数据库。

详细见文档
<A>http://docs.sequelizejs.com/en/v3/</a>

首先还是要安装好sequelize这个包。

然后还需要装好对应数据库所需要的一些包



```
$ npm install --save mysql // For both mysql and mariadb dialects
$ npm install --save sqlite3
$ npm install --save tedious // MSSQL
```

sequelize使用的时候需要创建一个实例，创建这个实例的时候便连接了数据库。

``` javascript
var sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'mysql'|'mariadb'|'sqlite'|'postgres'|'mssql',

  pool: {
    max: 5,
    min: 0,
    idle: 10000
  },

  // SQLite only
  storage: 'path/to/database.sqlite'
});

// Or you can simply use a connection uri
var sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');
```

### 创建模型
用define语句可以创建模型，在model文件夹下可以创建一个对应对象的model文件，然后export出去供其他人使用。

如图范例

``` javascript
const Sequelize = require('sequelize')
  , config = require('../config/db_config')
const sequelize = new Sequelize('bgm_db', null, null, config)
const subscriptionModel = {
  sub_id: {
    type: Sequelize.INTEGER,
    primaryKey: true
  },
  latest_episode_id: {
    type: Sequelize.INTEGER
  }
}

const Subscription = sequelize.define('subscription',subscriptionModel)
module.exports = Subscription
```