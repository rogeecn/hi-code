---
title: Go语言 channels 教程
date: 2018-01-26 00:00:00
tags: ["Go"]
abbrlink: course-of-golang-channels
img: ""
comments: false
---

Go语言内置了书写并发程序的工具。将go声明放到一个需调用的函数之前，在相同地址空间调用运行这个函数，这样该函数执行时便会作为一个独立的并发线程。这种线程在Go语言中称作goroutine。在这里我要提一下，并发并不总是意味着并行。Goroutines是指在硬件允许情况下创建能够并行执行程序的架构。这是这个主题的一次讨论：并发不是并行。

让我们从一个例子开始：

```go
func main() {
     // Start a goroutine and execute println concurrently
     go println("goroutine message")
     println("main function message")
}
```

这段程序将输出main function messageand 或者goroutine message。我说“ 或者”是因为催生的goroutine有一些特点。当你运行一个goroutine时，调用的代码（在我们的例子里它是main函数）不等待goroutine完成，而是继续往下运行。在调用完println后main函数结束了它的执行，在Go语言里这意味着这个程序及所有催生的goroutines停止执行。但是，在这个发生之前，goroutine可能已经完成了其代码的执行并输出了goroutine message字符。

你明白这些后必须有方法来避免这种情况。这就是Go语言中channels的作用。



## Channels 基础知识

Channels用来同步并发执行的函数并提供它们某种传值交流的机制。Channels的一些特性：通过channel传递的元素类型、容器（或缓冲区）和传递的方向由“<-”操作符指定。你可以使用内置函数 make分配一个channel:

```go
i := make(chan int)       // by default the capacity is 0
s := make(chan string, 3) // non-zero capacity

r := make(<-chan bool)          // can only read from
w := make(chan<- []os.FileInfo) // can only write to
```


Channels是一个第一类值（一个对象在运行期间被创建，可以当做一个参数被传递，从子函数返回或者可以被赋给一个变量。）可以像其他值那样在任何地方使用：作为一个结构元素，函数参数、函数返回值甚至另一个channel的类型：

```go
// a channel which:
//  - you can only write to
//  - holds another channel as its value
c := make(chan<- chan bool)

// function accepts a channel as a parameter
func readFromChannel(input <-chan string) {}

// function returns a channel
func getChannel() chan bool {
     b := make(chan bool)
     return b
}
```


在读、写channel的时候要格外注意 <- 操作符。它的位置关乎到channel变量的读写操作。下面的例子标明了它的使用方法，但我还是要提醒你，这段代码 并不会被完整地执行，原因我们后面再讲：

```go
func main() {
     c := make(chan int)
     c <- 42    // 写入channel
     val := <-c // 从channel中读取
     println(val)
}
```


现在我们知道了什么是channel，如何创建channel并且学了一些基础操作。现在让我们回到第一个示例，看看channel到底是如何帮助我们的。

```go
func main() {
     // 创建一个channel用以同步goroutine
     done := make(chan bool)

     // 在goroutine中执行输出操作
     go func() {
          println("goroutine message")

          // 告诉main函数执行完毕.
          // 这个channel在goroutine中是可见的
          // 因为它是在相同的地址空间执行的.
          done <- true
     }()

     println("main function message")
     <-done // 等待goroutine结束
}
```


这个程序将顺溜地打印2条信息。为什么呢？因为channel没有缓冲（我们没有指定其容量）。所有基于未缓冲的channel的的操作会将操作锁死直到输出和接收全部准备就绪。这就是为什么未缓冲channel也被称作同步（synchronous）。在我们的例子中，主函数中的操作符<-将会把程序锁死直到goroutine在channel中写入数据。因此程序只有在读取操作成功结束后才会终止。

为了避免存在一个channel的缓冲区所有读取操作都在没有锁定的情况下顺利完成（如果缓冲区是空的）并且写入操作也顺利结束（缓冲区不满），这样的channel被称作非同步的channel。下面是一个用来描述这两者区别的例子：

```go
func main() {
     message := make(chan string) // 无缓冲
     count := 3

     go func() {
          for i := 1; i <= count; i++ {
               fmt.Println("send message")
               message <- fmt.Sprintf("message %d", i)
          }
     }()

     time.Sleep(time.Second * 3)

     for i := 1; i <= count; i++ {
          fmt.Println(<-message)
     }
}
```


在这个例子中，输出信息是一个同步的channel，程序输出结果为：

```go
send message
// 等待3秒
message 1
send message
send message
message 2
message 3
```


正如你所看到的，在第一次goroutine中写入channel之后，其它在同一个channel中的写入操作都被锁住了，直到第一次读取操作执行完毕（大约3秒）。

现在我们提供一个缓冲区给输出信息的channel，例如：定义初始化行将被改为：message := make(chan string, 2)。这次程序输出将变为：

```go
send message
send message
send message
// 等待3秒
message 1
message 2
message 3
```


这里我们看到所有的写操作的执行都不会等待第一次对缓冲区的读取操作结束，channel允许储存所有的三条信息。通过修改channel容器，我们通过可以控制处理信息的总数达到限制系统输出的目的。

## 死锁

现在让我们回到前面那个没有成功运行的读/写操作示例：

```go
func main() {
     c := make(chan int)
     c <- 42    // 写入channel
     val := <-c // 读取channel
     println(val)
}
```


一旦运行此程序，你将得到以下错误：

```go
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
     /fullpathtofile/channelsio.go:5 +0x54
exit status 2
```


这个错误就是我们所知的死锁. 在这种情况下，两个goroutine互相等待对方释放资源，造成双方都无法继续运行。GO语言可以在运行时检测这种死锁并报错。这个错误是因为锁的自身特性产生的。

代码在次以单线程的方式运行，逐行运行。向channel写入的操作（c <- 42）会锁住整个程序的执行进程，因为在同步channel中的写操作只有在读取器准备就绪后才能成功执行。然而在这里，我们在写操作的下一行才创建了读取器。

为了使程序顺利执行，我们需要做如下改动：

```go
func main() {
     c := make(chan int)

     // 使写操作在另一个goroutine中执行。
     go func() {
        c <- 42
     }()
     val := <-c
     println(val)
}
```


## 范围化的channels 和channel的关闭

在前面的一个例子中，我们向channel发送了多条信息并读取它们，读取器部分的代码如下：

```go
for i := 1; i <= count; i++ {
     fmt.Println(<-message)
}
```


为了在执行读取操作的同时避免产生死锁，我们需要知道发送消息的确切数目，因为我们不能读取比写入条数还多的数据。但是这样很不方便，下面我们就提供了一个更为人性化的方法。

在Go语言中，存在一种称为范围表达式的代码，它允许程序反复声明数组、字符串、切片、图和channel，重复声明会一直持续到channel的关闭。请看下面的例子（虽然现在还不能执行）：

```go
func main() {
     message := make(chan string)
     count := 3

     go func() {
          for i := 1; i <= count; i++ {
               message <- fmt.Sprintf("message %d", i)
          }
     }()

     for msg := range message {
          fmt.Println(msg)
     }
}
```


很不幸的是，这段代码现在还不能运行。正如我们之前提到的，范围（range）只有等到channel关闭后才会运行。因此我们需要使用 close 函数关闭channel，程序就会变成下面这个样子：

```go
go func() {
     for i := 1; i <= count; i++ {
          message <- fmt.Sprintf("message %d", i)
     }
     close(message)
}()
```


关闭channel还有另外一个好处——被关闭的channel内的读取操作将不会引发锁，而是始终长生默认的对应channel类型的值：

```go
done := make(chan bool)
close(done)

//不会产生锁，打印两次false
//因为false是bool类型的默认值
println(<-done)
println(<-done)
```


这个特性可以被用于控制goroutine的同步，让我们再回顾一下之前同步的例子：

```go
func main() {
     done := make(chan bool)

     go func() {
          println("goroutine message")

          // 我们只关心被是否存在传送这个事实，而不是值的内容。
          done <- true
     }()

     println("main function message")
     <-done
}
```


在这里，done channel仅仅被用于同步程序执行，而不是发送数据。再举一个类似的例子：

```go
func main() {
     // 与数据内容无关
     done := make(chan struct{})

     go func() {
          println("goroutine message")

          // 发送信号"I'm done"
          close(done)
     }()

     println("main function message")
     <-done
}
```


我们关闭了goroutine中的channel，读取操作不会产生锁，因此主函数可以继续执行下去。

## 多channel模式和channel的选择

在真正的项目开发中，你可能需要多个goroutine和channel。当各部分的独立性越强，他们之间也就越需要高效的同步措施。让我们看个略微复杂的例子：

```go
func getMessagesChannel(msg string, delay time.Duration) <-chan string {
     c := make(chan string)
     go func() {
          for i := 1; i <= 3; i++ {
               c <- fmt.Sprintf("%s %d", msg, i)
               // 在发送信息前等待
               time.Sleep(time.Millisecond * delay)
          }
     }()
     return c
}

func main() {
     c1 := getMessagesChannel("first", 300)
     c2 := getMessagesChannel("second", 150)
     c3 := getMessagesChannel("third", 10)

     for i := 1; i <= 3; i++ {
          println(<-c1)
          println(<-c2)
          println(<-c3)
     }
}
```


这里我们创建了一个方法，用来创建channel并定义了一个goroutine使之在一此调用中向channel发送三条信息。我们看到，c3理应是最后一次channel调用，所以它的输出信息应该在其它信息之前。但是我们得到的却是如下输出：

```go
first 1
second 1
third 1
first 2
second 2
third 2
first 3
second 3
third 3
```


显然我们成功输出了所有的信息，这是因为第一个channel中的读取操作在每个循环声明中被锁住300毫秒，其它操作必须随之进入等待状态。而我们期望的却是从所有channel中尽快读取信息。

我们可以使用select 在多个channel之间进行选择。这种选择类似于普通的switch，但是所有的情况在这里都是数值传递操作（读/写）。即使操作数增加，程序也不会在更多的锁下运行。因此，如果想要达到我们之前的目的，我们可以这么改写程序：

```go
for i := 1; i <= 9; i++ {
     select {
     case msg := <-c1:
          println(msg)
     case msg := <-c2:
          println(msg)
     case msg := <-c3:
          println(msg)
     }
}
```


注意循环中的9这个数：每个channel存在三个写操作，这就是为什么这里需要9次循环的原因。在一般的守护进程中，我们可以使用无限循环执行选择操作，但如果我在这里那么做了，那我们将得到一个死锁：

```go
first 1
second 1
third 1 // 这个channel将不会等待其他channel
third 2
third 3
second 2
first 2
second 3
first 3
```


## 总结

channel是Go语言中颇为有趣的一个机制。但是在高效地使用它们之前你必须搞清楚他们是如何工作的。我试图在本文中对channel做出最基础的解释，如果你想要更深入地学习这个机制，我建议你阅读以下文章：

*   [并发不是并行](http://blog.golang.org/concurrency-is-not-parallelism) - Rob Pike 之前我们有提到过
*   [Go语言并发模式——初级篇](http://www.youtube.com/watch?v=f6kdp27TYZs)
*   [Go语言并发模式——高级篇](http://www.youtube.com/watch?v=QDDwwePbDtw)
