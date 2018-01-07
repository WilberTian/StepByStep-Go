Go在语⾔层⾯对并发编程提供⽀持，⼀种类似协程，称作`goroutine`的机制。



### `goroutine`相关概念

Go语言中并发指的是让某个函数独立于其他函数运行的能力，一个`goroutine`就是一个独立的工作单元，Go的`runtime`（运行时）会在逻辑处理器上调度这些`goroutine`，一个逻辑处理器绑定一个操作系统线程，所以说goroutine不是线程，它是由Go语言运行时本身的算法实现的一种类似协程的机制。

下面看看一些基本的概念：

概念           |  说明
--------------|-------------
逻辑处理器      |  执行创建的`goroutine`，绑定一个操作系统线程
调度器         |  Go运行时中的，分配`goroutine`给不同的逻辑处理器
全局运行队列    |  所有刚创建的`goroutine`都会放到这里
本地运行队列    |  逻辑处理器的`goroutine`队列


当创建一个`goroutine`后，会先存放在全局运行队列中，等待Go调度器进行调度，然后把`goroutine`分配给逻辑处理器，并放到逻辑处理器对应的本地运行队列中，最终等着被逻辑处理器执行即可。

这一套管理、调度、执行`goroutine`的方式称之为Go的并发。



### `goroutine`的使用

上面介绍了很多`goroutine`的概念，下面看看具体的使用，使用`goroutine`很简单，只需要在函数之前加上`go`关键字：

    func main() {
        var wg sync.WaitGroup
        wg.Add(2)

        go func() {
            defer wg.Done()
            fmt.Println("B")
        }()

        go func() {
            defer wg.Done()
            fmt.Println("C")
        }()

        fmt.Println("A")

        wg.Wait()
    }

因为进程退出时不会等待`goroutine`结束，为了能够看到`goroutine`的执行结果，例子中使用了`sync.WaitGroup`进行了同步。多次使用例子可以看到不同的结果，说明调度器不能保证多`goroutine`执⾏次序。

对于多核的CPU，默认情况下，Go默认是给每个可用的物理处理器都分配一个逻辑处理器，可使⽤环境变量或标准库函数`runtime.GOMAXPROCS`设置逻辑处理器的个数，让调度器⽤多个线程实现多核并⾏，⽽不仅仅是并发。

看下面的例子：

    import (
        "fmt"
        "sync"
        "time"
        "strconv"
        "math"
        "runtime"
    )

    func sum(id int) {
        var x int64
        for i := 0; i < math.MaxUint32; i++ {
            x += int64(i)
        }
        println(id, x)
    }

    func printTimestamp() int64 {
        t := time.Now()
        timestamp := strconv.FormatInt(t.UTC().UnixNano(), 10)
        
        int64, _ := strconv.ParseInt(timestamp, 10, 64) 

        return int64
    }

    func main() {
        // runtime.GOMAXPROCS(1)
        
        wg := new(sync.WaitGroup)
        wg.Add(2)

        start := printTimestamp()

        for i := 0; i < 2; i++ {
            go func(id int) {
                defer wg.Done()
                sum(id)
            }(i)
        }

        wg.Wait()

        end := printTimestamp()

        fmt.Println(end - start)
    }

当代码中使用`runtime.GOMAXPROCS(1)`设置后，两个`goroutine`**并发的**在同一个逻辑处理器上运行；当注释掉`runtime.GOMAXPROCS(1)`设置后，Go默认是给每个可用的物理处理器都分配一个逻辑处理器，两个`goroutine`就在不同逻辑处理器上并行的运行。对比下面的代码运行结果：

一个逻辑处理器并发：

    1 9223372030412324865
    0 9223372030412324865
    3814525864

多个逻辑处理器并行

    0 9223372030412324865
    1 9223372030412324865
    1778489366


对于逻辑处理器的个数，不是越多越好，要根据电脑的实际物理核数，如果不是多核的，设置再多的逻辑处理器个数也没用。如果需要设置，可以通过下面代码：

    runtime.GOMAXPROCS(runtime.NumCPU())




### `goroutine`的操作

通过调⽤`runtime.Goexit`可以⽴即终⽌当前`goroutine`，调度器会确保所有已注册`defer`延迟调⽤被执⾏。看一个例子：

    func main() {
        wg := new(sync.WaitGroup)
        wg.Add(1)

        go func() {
            defer wg.Done()
            defer fmt.Println("A.defer")

            func() {
                defer fmt.Println("B.defer")
                runtime.Goexit() 
                fmt.Println("B")     
            }()

            fmt.Println("A") 
        }()

        wg.Wait()
    }

代码输出为：

    B.defer
    A.defer

对于`goroutine`，可以通过`Gosched`让出底层线程，将当前`goroutine`暂停，放回队列等待下次被调度执⾏。看一个例子：

    func main() {
        runtime.GOMAXPROCS(1)

        wg := new(sync.WaitGroup)
        wg.Add(2)

        go func() {
            defer wg.Done()
            for i := 0; i < 6; i++ {
                fmt.Println(i)

                if i == 3 {
                    runtime.Gosched()
                }
            }
        }()

        go func() {
            defer wg.Done()
            runtime.Gosched()
            fmt.Println("Hello, World!")
        }()

        wg.Wait()
    }

代码输出为：

    0
    1
    2
    3
    Hello, World!
    4
    5

这个例子中，一定要注意`runtime.GOMAXPROCS(1)`的设置，保证两个`goroutine`是**并发的**执行。

