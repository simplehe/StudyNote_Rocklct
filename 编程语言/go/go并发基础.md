# go并发基础
Go主要提供了 **两种同步机制** 来保证多线程之间共享数据的一致性和安全性。

一种是传统的基于锁的同步机制，这种机制的使用方式跟其他语言差不多。提供了一套基本的同步工具比如：锁,条件变量,原子操作等等。

另一种是Go提供的一个新的同步机制：Channel。一个Channel可以类比为不同的go线程之间传递数据的管道。Go中的channel无论是实现机制还是使用场景都和Java中的BlockingQueue很接近。Go的这种并发模型，**也被人称为CSP（communicating sequential processes）并发模型。**

Go的并发编程中有一个很经典的思想：**不要以共享内存的方式去通信，而要以通信的方式去共享内存。** 在Go语言中并不鼓励用锁保护共享状态的方式在不同的Goroutine中分享信息(以共享内存的方式去通信)。**而是鼓励通过channel将共享状态或共享状态的变化在各个Goroutine之间传递（以通信的方式去共享内存）**，这样同样能像用锁一样保证在同一的时间只有一个Goroutine访问共享状态。

下面分别介绍go提供的两种同步机制。

## CSP并发模型

### goroutine
goroutine 是一个普通的函数，只是需要使用关键字go 作为开头。Go语言允许使用go语句开启一个新的 **运行期线程**。

``` go
func main() {
    go fmt.Println("Hello from another goroutine")
    fmt.Println("Hello from main goroutine")

    // 至此，程序运行结束，
    // 所有活跃的goroutine被杀死
  }
```

当一个程序启动时，其主函数即在一个单独的goroutine中运行，我们叫它main goroutine。新的goroutine会用go语句来创建。


**需要注意的一点就是，当主线程运行完毕后，所有goroutine都会停止**，所以假如上述main函数运行的快，那就会只输出一个hello from main,如果goroutine运行得足够快，那可能会输出hello from another和hello from main。

既然goroutine存在这种机制，那么当然就需要一种机制来确保goroutine之间的通信，来进行同步。**Channels就是用来做到这一点的。**

### Channels
如果说goroutine是Go程序的并发体的话，那么channels就是它们之间的通信机制。一个channels可以让一个goroutine通过它给另一个goroutine发送值信息。

每个channel都有一个特殊的类型，也就是channels可发送数据的类型。一个可以发送int类型数据的channel一般写为chan int。channel类型一般使用内置的make函数创建，make函数经常用于创建map，slice，channel。

``` go
ch := make(chan int) // ch has type 'chan int'
```

一个channel有发送和接受两个主要操作，发送和接收两个操作都是用<-运算符。

``` go
ch <- v    // 将 v 发送至信道 ch。
v := <-ch  // 从 ch 接收值并赋予 v。
```

**默认情况下，发送和接收操作在另一端准备好之前都会阻塞。这使得 Go 程可以在没有显式的锁或竞态变量的情况下进行同步。**

所以利用信道，很方便就能实现Go程同步。

```
func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // 将和送入 c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // 从 c 中接收

	fmt.Println(x, y, x+y)
}
```

上述代码，启动两个goroutine去计算数组的前半部分和后半部分的和。后面的x，y变量阻塞在信道上去获取两个值，两个值都获取到以后输出和。

#### 带缓冲的Channels
一个基于无缓存Channels的发送操作将导致发送者goroutine阻塞，直到另一个goroutine在相同的Channels上执行接收操作，当发送的值通过Channels成功传输之后，两个goroutine可以继续执行后面的语句。

而信道可以是 带缓冲的。将缓冲长度作为第二个参数提供给 make 来初始化一个带缓冲的信道：

``` go
ch := make(chan int, 100)
```

仅当信道的缓冲区填满后，向其发送数据时才会阻塞。当缓冲区为空时，接受方会阻塞。需要注意的是channel的第二个参数是缓冲里可容纳值得个数，不是字节数。

当所有goroutine都卡在channel上的时候，就会陷入死锁。

#### range和close
发送者可通过 close 关闭一个信道来表示没有需要发送的值了。接收者可以通过为接收表达式分配第二个参数来测试信道是否被关闭：

```
v, ok := <-ch
```

若没有值可以接收且信道已被关闭，那么在执行完之后 ok 会被设置为 false。

**注意：只有发送者才能关闭信道，而接收者不能。**

而range，则用来不停读取Channel里的值，直到信道被关闭：

```
for i := range c
```

信道与文件不同，通常情况下无需关闭它们。只有在必须告诉接收者不再有值需要发送的时候才有必要关闭，例如终止一个 range 循环。

#### select语句
select 语句使一个 Go 程可以等待多个通信操作。

select 会阻塞到某个分支可以继续执行为止，这时就会执行该分支。当多个分支都准备好时会随机选择一个执行。

看一个打印斐波那契数列的例子：

``` go
func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}

```

main函数中创建了两个信道，开启了一个go程，这个go程首先会读10次c信道，然后才再往quit丢一个0值。同时main程还会执行斐波那契数列函数，传递两个信道进去。fibonacci函数调用select语句，如果c信道是空，就不停传递x值，如果quit信道有值就输出并且返回。

由于一开始quit信道为空，select则会不断往c信道里放x的值，直到func的goroutine循环到10次不再读取c信道的值。此时case c的分支会被阻塞，quit被放入0以后，main程退出。


当 select 中的其它分支都没有准备好时，default 分支就会执行。为了在尝试发送或者接收时不发生阻塞，可使用 default 分支：

``` go
select {
case i := <-c:
    // 使用 i
default:
    // 从 c 中接收会阻塞时执行
}
```

## 基于共享变量的并发
普通的线程并发模型，就是像Java、C++、或者Python，他们线程间通信都是通过共享内存的方式来进行的。非常典型的方式就是，在访问共享数据（例如数组、Map、或者某个结构体或对象）的时候，通过锁来访问，因此，在很多时候，衍生出一种方便操作的数据结构，叫做“线程安全的数据结构”。例如Java提供的包”java.util.concurrent”中的数据结构。Go中也实现了传统的线程并发模型。

### sync.Mutex
Go 标准库中提供了 sync.Mutex 互斥锁类型及其两个方法：

 - Lock
 - Unlock

我们可以通过在代码前调用 Lock 方法，在代码后调用 Unlock 方法来保证一段代码的互斥执行。

``` go
// SafeCounter 的并发使用是安全的。
type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

// Inc 增加给定 key 的计数器的值。
func (c *SafeCounter) Inc(key string) {
	c.mux.Lock()
	// Lock 之后同一时刻只有一个 goroutine 能访问 c.v
	c.v[key]++
	c.mux.Unlock()
}
```

上面代码定义了一个SafeCounter结构体，然后定义了这个结构体对应的方法Inc。这个Inc方法加了锁，是线程安全的。

Lock和UnLock之间的的代码段，就是我们所说的 **临界区**。在复杂的临界区应用中，我们有时候往往会忘记调用UnLock，这时我们可以使用defer关键字。defer和go一样都是Go语言提供的关键字。defer用于资源的释放，会在函数返回之前进行调用。这样我们就从“总要记得在函数返回之后或者发生错误返回时得调用一次Unlock”这种状态中获得了解放。Go帮助我们自动完成这些事情。

``` go
func Balance() int {
  mu.Lock()
  defer mu.Unlock()
  return balance
}
```

解锁会延迟到作用域后执行，这大大提高了代码的整洁性。

### 读写锁sync.RWMutex
有时我们需要一种特殊类型的锁，其允许多个只读操作并行执行，但写操作会完全互斥。这种锁叫作“多读单写”锁(multiple readers, single writer lock)，Go语言提供的这样的锁是sync.RWMutex。

``` go
var mu sync.RWMutex
var balance int
func Balance() int {
    mu.RLock() // readers lock
    defer mu.RUnlock()
    return balance
}
```

以上函数用来读取余额，允许多个读go程并行执行。

### sync.WaitGroup
sync.WaitGroup能够一直等到所有的goroutine执行完成，并且阻塞主线程的执行，直到所有的goroutine执行完成。

``` go
func main() {
    var wg sync.WaitGroup
    wg.Add(2) // 因为有两个动作，所以增加2个计数
    go func() {
        fmt.Println("Goroutine 1")
        wg.Done() // 操作完成，减少一个计数
    }()
    go func() {
        fmt.Println("Goroutine 2")
        wg.Done() // 操作完成，减少一个计数
    }()
    wg.Wait() // 等待，直到计数为0
}
```

WaitGroup内部实现了一个计数器，用来记录未完成的操作个数，它提供了三个方法，Add()用来添加计数。Done()用来在操作结束时调用，使计数减一。Wait()用来等待所有的操作结束，即计数变为0，该函数会在计数不为0时等待，在计数为0时立即返回。这种机制就有点像Java的CountDownLatch

### 总结
其实两种并发方案并不能说哪种一定更好，不同情境下应当使用合适的方案来完成。

## 参考
《学习Go语言》---Miek Gieben

[《Go语言圣经》](https://legacy.gitbook.com/book/yar999/gopl-zh/details)

[Go并发编程基础](http://blog.xiayf.cn/2015/05/20/fundamentals-of-concurrent-programming/)

[Go指南](https://tour.go-zh.org/list)

[GO并发机制](https://github.com/k2huang/blogpost/blob/master/golang/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/%E5%B9%B6%E5%8F%91%E6%9C%BA%E5%88%B6/Go%E5%B9%B6%E5%8F%91%E6%9C%BA%E5%88%B6.md)
