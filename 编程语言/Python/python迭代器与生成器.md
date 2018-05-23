## 迭代器与生成器

python中，**两种** 数据类型可以作用于for：

一种是集合数据类型，**如list、tuple、dict、set、str** 等；

一种是 **generator**，包括 **生成器和带yield的generator function**。

这些可以直接作用于for循环的对象统称为 **可迭代对象：Iterable**。


### isinstance()

可以使用isinstance()判断一个对象是否是Iterable对象

``` python
from collections import Iterable
isinstance([], Iterable) # True
# {} 'abc' 等都是可迭代对象

```

### next

而生成器不但可以作用于for循环，还可以被next()函数不断调用并返回下一个值，直到最后抛出StopIteration错误表示无法继续返回下一个值了。

可以被next()函数调用并不断返回下一个值的对象称为迭代器：Iterator。

**这里要注意，生成器都是Iterator对象，但list、dict、str虽然是Iterable，却不是Iterator。**

#### Iterator(迭代器)和Iterable(可迭代对象)的区别
这里要明确概念，Iterator代表的是 **数据流** ，可以把这个数据流看做是一个有序序列，但我们却 **不能提前知道序列的长度**，只能不断通过next()函数实现按需计算下一个数据，所以 **Iterator的计算是惰性的，只有在需要返回下一个数据时它才会计算**。

Iterator甚至可以表示一个无限大的数据流，例如全体自然数。而使用list是永远不可能存储全体自然数的。

把list、dict、str等Iterable变成Iterator可以使用iter()函数：

#### for本质

Python的for循环本质上就是通过不断调用next()函数实现的，例如

``` python

for x in [1, 2, 3, 4, 5]:
    pass

# 等价于

# 首先获得Iterator对象:
it = iter([1, 2, 3, 4, 5])
# 循环:
while True:
    try:
        # 获得下一个值:
        x = next(it)
    except StopIteration:
        # 遇到StopIteration就退出循环
        break


```

### 生成器generator
通过列表生成式，我们可以直接创建一个列表。但是，受到内存限制，列表容量肯定是有限的。

所以，如果列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢？这样就不必创建完整的list，从而节省大量的空间。在Python中，这种一边循环一边计算的机制，称为生成器：generator。**毫无疑问，generator就是一个iterator**

要创建一个generator，有很多种方法。第一种方法很简单，只要把一个列表生成式的[]改成()，就创建了一个generator：

```
>>> L = [x * x for x in range(10)]
>>> L
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
>>> g = (x * x for x in range(10))
>>> g
<generator object <genexpr> at 0x1022ef630>
```

generator保存的是算法，每次调用next(g)，就计算出g的下一个元素的值，直到计算到最后一个元素，没有更多的元素时，抛出StopIteration的错误。

由于不断调用next(g)实在是太变态了，正确的方法是 **使用for循环**，因为generator也是可迭代对象：

#### yield关键字
定义generator的另一种方法。就是yield关键字

``` python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
    return 'done'
```

变成generator的函数，在每次调用next()的时候执行，遇到yield语句返回，再次执行时从上次返回的yield语句处继续执行。

``` python
>>> g = fib(6)
>>> g
<generator object fib at 0x1022ef948>
```
