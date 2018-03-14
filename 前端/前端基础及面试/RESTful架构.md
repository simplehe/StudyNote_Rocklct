## RESTful架构

如果一个架构符合REST原则，就称它为RESTful架构

（最新笔记见分布式通信系统高级抽象笔记）

### REST关键原则

实际上REST的关键原则有

- 为所有“事物”定义ID
- 将所有事物链接在一起
- 使用标准方法
- 资源多重表述
- 无状态通信

然而基于HTTP进行总结后，可概括为以下三点。

- 每一个URI代表一种资源
- 客户端和服务端之间，传递这种资源的某种表现层（比如xml，json）
- 客户端通过HTTP动词，对服务端资源进行操作，实现表现层（json，xml）状态转化。

基于RESTful 架构的API是目前比较成熟的一套互联网应用程序API设计理论，其概念必须掌握。

归纳以上三点，以下是设计api时候的一些例子

- GET /products : will return the list of all products
- POST /products : will add a product to the collection
- GET /products/4 : will retrieve product #4PATCH/
- PUT /products/4 : will update product #4

动词和url进行搭配，url里可以嵌套versioning
