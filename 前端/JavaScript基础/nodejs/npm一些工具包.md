## 整理一些npm常用的工具包


### lodash
lodash一开始是Underscore.js库的一个fork，因为和其他(Underscore.js的)贡献者意见相左。John-David Dalton的最初目标，是提供更多“一致的跨浏览器行为……，并改善性能”。之后，该项目在现有成功的基础之上取得了更大的成果。最近lodash也发布了3.5版，成为了npm包仓库中依赖最多的库。它正在摆脱屌丝身份，成为开发者的常规的选择之一。

lodash主要使用了延迟计算，使得lodash其性能远远超过Underscore。在lodash中延迟计算意味着在我们的链式方法在显示或隐式的value()调用之前是不会执行的。由于这种执行的延后，因此lodash可以进行shortcut fusion这样的优化，通过合并链式iteratee大大降低迭代的次数。从而大大提供其执行性能。