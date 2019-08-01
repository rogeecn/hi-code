---
title: 在 Go 语言中，正确的使用并发
date: 2018-01-25 00:00:00
tags: ["Go"]
abbrlink: use-channel-correctly-in-golang
img: ""
comments: false
---

Glyph Lefkowitz最近写了一篇启蒙文章，其中他详细的说明了一些关于开发高并发软件的挑战，如果你开发软件但是没有阅读这篇问题，那么我建议你阅读一篇。这是一篇非常好的文章，现代软件工程应该拥有的丰富智慧。

从多个花絮中提取，但是如果我斗胆提出主要观点的总结，其内容就是：抢占式多任务和一般共享状态结合导致软件开发过程不可管理的复杂性， 开发人员可能更喜欢保持自己的一些理智以此避免这种不可管理的复杂性。抢占式调度对于哪些真正的并行任务是好的，但是当可变状态通过多并发线程共享时，明确的多任务合作更招人喜欢 。

尽管合作多任务，你的代码仍有可能是复杂的，它只是有机会保持可管理下一定的复杂性。当控制转移是明确一个代码阅读者至少有一些可见的迹象表明事情可能脱离正轨。没有明确标记每个新阶段是潜在的地雷：“如果这个操作不是原子操作，最后出现什么情况？”那么在每个命令之间的空间变成无尽的空间黑洞，可怕的Heisenbugs出现

在过去的一年多，尽管在Heka上的工作（一个高性能数据、日志和指标处理引擎）已大多数使用GO语言开发。Go的亮点之一就是语言本身有一些非常有用的并发原语。但是Go的并发性能怎么样，需要通过支持本地推理的鼓励代码镜头观察。

并非事实都是好的。所有的Goroutine访问相同的共享内存空间，状态默认可变，但是Go的调度程序不保证在上下文选择过程中的准确性。在单核设置中，Go的运行时间进入“隐式协同工作”一类， 在Glyph中经常提到的异步程序模型列表选择4。 当Goroutine能够在多核系统中并行运行，世事难料。

Go不可能保护你，但是并不意味着你不能采取措施保护自己。在写代码过程中通过使用一些Go提供的原语，可最小化相关的抢占式调度产生的异常行为。请看下面Glyph示例“账号转换”代码段中Go接口（忽略哪些不易于最终存储定点小数的浮点数）



```
func Transfer(amount float64, payer, payee *Account,
    server SomeServerType) error {
    if payer.Balance() < amount {
        return errors.New("Insufficient funds")
    }
    log.Printf("%s has sufficient funds", payer)
    payee.Deposit(amount)
    log.Printf("%s received payment", payee)
    payer.Withdraw(amount)
    log.Printf("%s made payment", payer)
    server.UpdateBalances(payer, payee) // Assume this is magic and always works.
    return nil
}
```
这明显的是不安全的，如果从多个goroutine中调用的话，因为它们可能并发的从存款调度中得到相同的结果，然后一起请求更多的已取消调用的存款变量。最好是代码中危险部分不会被多goroutine执行。在此一种方式实现了该功能：
```type transfer struct {
    payer *Account
    payee *Account
    amount float64
}
var xferChan = make(chan *transfer)
var errChan = make(chan error)
func init() {
    go transferLoop()
}
func transferLoop() {
    for xfer := range xferChan {
        if xfer.payer.Balance < xfer.amount {
            errChan <- errors.New("Insufficient funds")
            continue
        }
        log.Printf("%s has sufficient funds", xfer.payer)
        xfer.payee.Deposit(xfer.amount)
        log.Printf("%s received payment", xfer.payee)
        xfer.payer.Withdraw(xfer.amount)
        log.Printf("%s made payment", xfer.payer)
        errChan <- nil
    }
}
func Transfer(amount float64, payer, payee *Account,
    server SomeServerType) error {
    xfer := &transfer{
        payer: payer,
        payee: payee,
        amount: amount,
    }
    xferChan <- xfer
    err := <-errChan
    if err == nil  {
        server.UpdateBalances(payer, payee) // Still magic.
    }
    return err
}
```

这里有更多代码，但是我们通过实现一个微不足道的事件循环消除并发问题。当代码首次执行时，它激活一个goroutine运行循环。转发请求为了此目的而传递入一个新创建的通道。结果经由一个错误通道返回到循环外部。因为通道不是缓冲的，它们加锁，并且通过Transfer函数无论多个并发的转发请求怎么进，它们都将通过单一的运行事件循环被持续的服务。

上面的代码看起来有点别扭，也许吧. 对于这样一个简单的场景一个互斥锁（mutex）也许会是一个更好的选择，但是我正要尝试去证明的是可以向一个go例程应用隔离状态操作. 即使稍稍有点尴尬，但是对于大多数需求而言它的表现已经足够好了，并且它工作起来，甚至使用了最简单的账号结构实现:

```go
type Account struct {
    balance float64
}
func (a *Account) Balance() float64 {
    return a.balance
}
func (a *Account) Deposit(amount float64) {
    log.Printf("depositing: %f", amount)
    a.balance += amount
}
func (a *Account) Withdraw(amount float64) {
    log.Printf("withdrawing: %f", amount)
    a.balance -= amount
}
```
不过如此笨拙的账户实现看起来会有点天真. 通过不让任何大于当前平衡的撤回操作执行，从而让账户结构自身提供一些保护也许更起作用。那如果我们把撤回函数变成下面这个样子会怎么样呢?:

```
func (a *Account) Withdraw(amount float64) {
    if amount > a.balance {
        log.Println("Insufficient funds")
        return
    }
    log.Printf("withdrawing: %f", amount)
    a.balance -= amount
}
```
不幸的是，这个代码患有和我们原来的 Transfer 实现相同的问题。并发执行或不幸的上下文切换意味着我们可能以负平衡结束。幸运的是，内部的事件循环理念应用在这里同样很好，甚至更好，因为事件循环 goroutine 可以与每个个人账户结构实例很好的耦合。这里有一个例子说明这一点：

```type Account struct {
    balance float64
    deltaChan chan float64
    balanceChan chan float64
    errChan chan error
}
func NewAccount(balance float64) (a *Account) {
    a = &Account{
        balance:     balance,
        deltaChan:   make(chan float64),
        balanceChan: make(chan float64),
        errChan:     make(chan error),
    }
    go a.run()
    return
}
func (a *Account) Balance() float64 {
    return <-a.balanceChan
}
func (a *Account) Deposit(amount float64) error {
    a.deltaChan <- amount
    return <-a.errChan
}
func (a *Account) Withdraw(amount float64) error {
    a.deltaChan <- -amount
    return <-a.errChan
}
func (a *Account) applyDelta(amount float64) error {
    newBalance := a.balance + amount
    if newBalance < 0 {
        return errors.New("Insufficient funds")
    }
    a.balance = newBalance
    return nil
}
func (a *Account) run() {
    var delta float64
    for {
        select {
        case delta = <-a.deltaChan:
            a.errChan <- a.applyDelta(delta)
        case a.balanceChan <- a.balance:
            // Do nothing, we've accomplished our goal w/ the channel put.
        }
    }
}
```

这个API略有不同，Deposit 和 Withdraw 方法现在都返回了错误。它们并非直接处理它们的请求，而是把账户余额的调整量放入 deltaChan，在 run 方法运行时的事件循环中访问 deltaChan。同样的，Balance 方法通过阻塞不断地在事件循环中请求数据，直到它通过 balanceChan 接收到一个值。

须注意的要点是上述的代码，所有对结构内部数据值得直接访问和修改都是有事件循环触发的 *within* 代码来完成的. 如果公共 API 调用表现良好并且只使用给出的渠道同数据进行交互的话, 那么不管对公共方法进行多少并发的调用，我们都知道在任意给定的时间只会有它们之中的一个方法得到处理. 我们的时间循环代码推理起来更加容易了很多.

该模式的核心是 Heke 的设计. 当Heka启动时，它会读取配置文件并且在它自己的go例程中启动每一个插件. 随着时钟信号、关闭通知和其它控制信号，数据经由通道被送入插件中. 这样就鼓励了插件作者使用一种想上述事例那样的 事件循环类型的架构 来实现插件的功能.

再次，GO不会保护你自己. 写一个同其内部数据管理和主题有争议的条件保持松耦合的Heka插件（或者任何架构）是完全可能的. 但是有一些需要注意的小地方，还有Go的争议探测器的自由应用程序，你可以编写的代码其行为可以预测，甚至在抢占式调度的门面代码中.
