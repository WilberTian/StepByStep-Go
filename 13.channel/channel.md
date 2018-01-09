在Go语言中，多个`goroutine`可以并发（并行）的执行，如果多个`goroutine`之间通信就需要用到`channel`。

`channel`实现了多个`goroutine`之间发送和接收数据，从而实现数据共享。




### `channel`的使用

在Go中，可以使用关键字`chan`声明一个`channel`，同时指定`channel`中发送和接收数据的类型。

例如下面声明了一个可以发送、接收`int`类型的`channel`：

	var ch chan int


`channel`和`slice`、`map`一样，是引用类型，声明之后不能直接使用，需要通过内置的`make`函数声明初始化：

	// 无缓冲的`channel`
	ch := make(chan int)

	// 有缓冲的`channel`
	ch := make(chan int, 2)

通过`make`函数声明初始化，可以通过第二个参数指定`channel`的大小（可以存放数据的数量）；当大小为0的时候，就是一个无缓冲的`channel`。

通过`close`这个内部函数，可以关闭`channel`，向一个已经关闭的`channel`发送数据会引起painc异常。

	close(ch)




### 无缓冲的`channel`

首先看看无缓冲的`channel`，由于无缓冲区的`channel`没有保存数据的能力，所以发送`goroutine`和接收`gouroutine`必须是**同步的**方式工作。

看一个例子：

	func main() {
		data := make(chan int)  
		exit := make(chan bool) 
		go func() {
			for d := range data { 	// 从队列迭代接收数据，直到 close 
				fmt.Println(d)
			}

			fmt.Println("recv over.")
			
			exit <- true 
		}()

		data <- 1 
		data <- 2
		data <- 3
		close(data) 
		
		fmt.Println("send over.")
		
		<-exit 
	}

代码输出为：

	1
	2
	send over.
	3
	recv over.

在`channel`的使用过程中，数据的发送和接收通过`<-`操作符完成：

- 发送：`channel`对象在`<-`操作符左边
- 接收：`channel`对象在`<-`操作符右边

另外，**可以通过`range`以及`ok-idiom`模式来判断`channel`是否已经关闭**。




### 有缓冲的`channel`

对于有缓冲的`channel`，缓冲区是一个队列，这个队列的最大容量就是`make`函数创建通道时的第二个参数指定的。

向有缓冲的`channel`发送操作就是向缓冲区队列的尾部插入数据，接收操作则是从缓冲区队列的头部删除元素，并返回这个删除的元素。

当多个`goroutine`使用有缓冲的`channel`进行通信时，不要求发送方和接收方同步工作，多个`goroutine`之间是以异步⽅式工作的。但是如果缓冲区已满，那么发送被阻塞；缓冲区为空，则接收被阻塞。

看一有缓冲`channel`的例子：

	func main() {
		data := make(chan int, 3) 	// 缓冲区可以存储 3 个元素
		exit := make(chan bool)

		data <- 1 					// 在缓冲区未满前，不会阻塞
		data <- 2
		data <- 3

		go func() {
			for d := range data { 	// 在缓冲区未空前，不会阻塞
				fmt.Println(d)
			}

			exit <- true
		}()

		data <- 4 
		data <- 5

		close(data)

		<-exit
	}

另外，对于有缓冲的`channel`，可以使用`cap`和`len`函数分别获取`channel`的容量和`channel`里已有的数据长度。



### `select`

如果需要同时处理多个`channel`，可使⽤`select`语句，它随机选择⼀个可⽤`channel`做收发操作，或者执⾏`default case`。

看一个例子：

func main() {
	a, b := make(chan int, 2), make(chan int)

	go func() {
		v, ok, s := 0, false, ""

		for {
			select { 				// 随机选择可⽤ channel，接收数据。
			case v, ok = <-a:
				s = "a"
			case v, ok = <-b:
				s = "b"
			}

			if ok {
				fmt.Println(s, v)
			} else {
				os.Exit(0)
			}
		}
	}()

	for i := 0; i < 5; i++ {
		select { 					// 随机选择可⽤ channel，发送数据。
		case a <- i:
		case b <- i:
		}
	}

	close(a)

	select {} 						// 没有可⽤ channel，阻塞 main goroutine。
}


对于有`default`的`select`，当所有的`channel`都不可用时，就执行`default`：

	func main() {
		ch1 := make(chan int, 1)
		ch2 := make(chan int, 1)

		select {
		case <-ch1:
			fmt.Println("ch1 pop one element")
		case <-ch2:
			fmt.Println("ch2 pop one element")
		default:
			fmt.Println("default")
		}
	}

代码输出：

	default

### `select`的典型用法

使用`select`实现`timeout`机制：

	func main() {
		w := make(chan bool)
		c := make(chan int)

		go func() {
			select {
			case v := <-c:
				fmt.Println(v)
			case <-time.After(time.Second * 3):
				fmt.Println("timeout.")
			}
			w <- true
		}()

		// c <- 1 		// 注释掉该语句，引发 timeout。
		<-w
	}

使用`select`来检测`channel`是否已满：

	func main() {
		ch := make(chan int, 2)
		ch <- 1
		ch <- 2

		select {
		case ch <- 3:
		default:
			fmt.Println("channel is full!")
		}
	}

