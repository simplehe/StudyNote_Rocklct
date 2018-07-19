## go并发基础
Go并发主要围绕着两个概念，**goroutine和channel。**

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


**需要注意的一点就是，当主线程运行完毕后，所有goroutine都会停止**，所以假如上述main函数运行的快，那就会只输出一个hello from main,如果goroutine运行得足够快，那可能会输出hello from another和hello from main。不会出现hello from main在hello from another的前面。

既然goroutine存在这种机制，那么当然就需要一种机制来确保goroutine之间的通信，来进行同步。**Channels就是用来做到这一点。**

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


### 参考
《学习Go语言》---Miek Gieben

[《Go语言圣经》](https://legacy.gitbook.com/book/yar999/gopl-zh/details)

[Go并发编程基础](http://blog.xiayf.cn/2015/05/20/fundamentals-of-concurrent-programming/)

[Go指南](https://tour.go-zh.org/list)
