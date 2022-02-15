# The Go Memory Model

## Introduction

​	Go 内存模型指定了在何种条件下，一个 goroutine 中的变量读取可以保证观察到不同 goroutine 中对同一变量的写入所产生的值

## Advice

​	修改被多个 goroutine 同时访问的数据的程序必须序列化这种访问。

​	要序列化访问，请使用channel操作或其他同步原语（例如sync和sync/atomic包中的同步原语）保护数据。

## Happens Before

​	在单个 goroutine 中，读取和写入必须按照程序指定的顺序执行。也就是说，只有当重新排序不会改变语言规范所定义的 goroutine 内的行为时，编译器和处理器才可以重新排序在单个 goroutine 内执行的读取和写入。由于这种重新排序，一个 goroutine 观察到的执行顺序可能与另一个 goroutine 观察到的顺序不同。例如，如果一个 goroutine 执行 a = 1; b = 2;，另一个可能会观察到b的赋值先于a的赋值。

​	为了指定读取和写入的要求，我们定义了happens before（发生在...之前），在 Go 程序中执行内存操作的局部顺序(partial order)。如果事件 e1 发生在事件 e2 之前，那么我们说 e2 发生在 e1 之后。 此外，如果 e1 没有发生在 e2 之前并且没有发生在 e2 之后，那么我们说 e1 和 e2 同时发生。

​	在单个 goroutine 中，happens-before 顺序是程序表达的顺序。

​	如果以下两个条件都成立，则允许对变量 v 的读取 r 观察到对 v 的写入w：

- r并不先于w发生
- 在 w 之后但在 r 之前没有发生其他对 v 的 w' 写入。

​	为了保证变量 v 的读取 r 观察到对 v 的特定写入 w，请确保 w 是 r 观察到的唯一写入。 也就是说，如果以下两个都成立，则保证 r 可以观察到 w：

- w 发生在 r 之前。
- 对共享变量 v 的任何其他写入都发生在 w 之前或 r 之后。

​	这对条件比第一对强，它要求没有与 w 或 r 同时发生的其他写入。

​	在单个 goroutine 内，没有并发，所以两个定义是等价的：一个读 r 观察最近一次写 w 写入 v 的值。当多个 goroutine 访问一个共享变量 v 时，它们必须使用同步事件来建立 happens-before 确保读取遵守所需写入的条件。

​	使用 v 类型的零值初始化变量 v 的行为类似于在内存模型中的写入。

​	大于单个机器字的值的读取和写入表现为未指定顺序的多个机器字大小的操作。

## Synchronization

### Initialization

​	程序初始化在单个 goroutine 中运行，但该 goroutine 可能会创建其他并发运行的 goroutine。

​	如果包 p 导入包 q，q 的 init 函数的完成发生在 p 的任何函数开始之前。

​	函数 main.main 的启动发生在所有 init 函数完成之后。

### Goroutine creation

​	启动新 goroutine 的 go 语句发生在 goroutine 执行开始之前。

​	例如，在这个程序中：

```go
var a string
func f() {
	print(a)
}
func hello() {
	a = "hello, world"
	go f()
}
```

​	调用hello将会打印出"hello,world"在未来的某个时间点（可能会在hello返回之后）

### Goroutine destruction

​	goroutine 的退出不能保证在程序中的任何事件之前发生。例如，在这个程序中：

```go
var a string
func hello() {
	go func() { a = "hello" }()
	print(a)
}
```

​	对 a 的赋值之后没有任何同步机制，因此不能保证赋值被任何其他 goroutine 观察到。事实上，激进的编译器可能会删除整个 go 语句。

​	如果一个 goroutine 的结果必须被另一个 goroutine 观察到，那么使用诸如lock或channel通信之类的同步机制来建立相对顺序。

### Channel communication

​	channel通信是 goroutine 之间同步的主要方法。 特定channel上的每个发送都与来自该channel的相应接收匹配，通常发送和接受发生在在不同的 goroutine中。

​	channel上的发送发生在该channel的相应接收完成之前。

​	这个程序:

```go
var c = make(chan int, 10)
var a string
func f() {
	a = "hello, world"
	c <- 0
}
func main() {
	go f()
	<-c
	print(a)
}
```

​	保证打印“hello,world”。 对 a 的写入发生在 c 上的发送之前，这发生在 c 上的相应接收完成之前，而c完成接受发生在打印之前，保证了整体的相对顺序。

​	channel的关闭发生在接收返回零值之前，因为channel已关闭。

​	在前面的示例中，将 c <- 0 替换为 close(c) 会产生具有相同保证行为的程序。（即关闭channel也可以从channel中读出相应的消息，只不过是一个零值）

​	来自**无缓冲channel**的接收发生在该channel上的发送完成之前。

​	这个程序（如上所述，但交换了发送和接收语句并使用无缓冲channel）:

```go
var c = make(chan int)
var a string
func f() {
	a = "hello, world"
	<-c
}
func main() {
	go f()
	c <- 0
	print(a)
}
```

​	也保证打印“hello,world”。 对 a 的写入发生在 c 上的接收之前，这发生在 c 上的相应发送完成之前，这发生在打印之前。

​	如果channel被缓冲（例如，c = make(chan int, 1)），那么程序将不能保证打印“hello, world”。 （它可能会打印空字符串、崩溃或做其他事情。）

​	容量为 C 的channel上的第 k 次接收发生在该channel的第 k+C 次发送完成之前。

​	该规则将前面的规则推广到缓冲channel。它允许通过缓冲channel对计数信号量进行建模：通道中的item数量对应活跃使用的数量，channel的容量对应最大同时使用数量，发送item获取semaphore，接收item释放semaphore。这是限制并发的常用习语。

​	该程序为工作列表中的每个条目启动一个 goroutine，但这些 goroutine 使用限制通道进行协调，以确保一次最多运行三个工作函数。

```go
var limit = make(chan int, 3)
func main() {
	for _, w := range work {
		go func(w func()) {
			limit <- 1
			w()
			<-limit
		}(w)
	}
	select{}
}
```

### Locks

​	sync 包实现了两种锁数据类型，sync.Mutex 和sync.RWMutex。

​	对于任何 sync.Mutex 或 sync.RWMutex 变量 l 和 n < m(表示次数)，l.Unlock() 的第n次调用 发生在 l.Lock() 的第m次调用返回之前。

​	这个程序：

```go
var l sync.Mutex
var a string

func f() {
	a = "hello, world"
	l.Unlock()
}
func main() {
	l.Lock()
	go f()
	l.Lock()
	print(a)
}
```

​	保证打印“hello,world”。 对 l.Unlock() 的第一次调用（在 f 中）发生在对 l.Lock() 的第二次调用（在 main 中）返回之前，第二次调用Lock发生在打印之前。

### Once

​	sync 包通过使用 Once 类型为存在多个 goroutine 的情况下提供了一种安全的初始化机制。 多个线程可以执行once.Do(f) 用于特定的 f，但只有一个会运行 f()，其他调用会阻塞，直到 f() 返回。

​	从 once.Do(f) 对 f() 的单次调用发生（返回）在任何对 once.Do(f) 的调用返回之前。

​	在这个程序中:

```go
var a string
var once sync.Once
func setup() {
	a = "hello, world"
}
func doprint() {
	once.Do(setup)
	print(a)
}
func twoprint() {
	go doprint()
	go doprint()
}
```

​	调用 twoprint 将只调用一次 setup 。 setup function将在打印调用之前完成。 结果将是“hello, world”将被打印两次。

## Incorrect synchronization

​	请注意，读取 r 可能会观察到与 r 同时发生的写入 w 写入的值。 即使发生这种情况，也并不意味着发生在 r 之后的读取会观察到发生在 w 之前的写入。

​	在这个程序中：

```go
var a, b int

func f() {
	a = 1
	b = 2
}
func g() {
	print(b)
	print(a)
}
func main() {
	go f()
	g()
}
```

​	可能会发生 g 打印 2 然后 0。

​	这一事实使一些常见的惯用语法无效。

​	双重检查锁定是一种避免同步开销的尝试。 例如，twoprint 程序可能被错误地写为：

```go
var a string
var done bool

func setup() {
  //一下两句可能会调换顺序
	a = "hello, world"
	done = true
}

func doprint() {
	if !done {
		once.Do(setup)
	}
	print(a)
}

func twoprint() {
	go doprint()
	go doprint()
}
```

​	但不能保证，在 doprint 中，观察到对done的写入意味着观察到对 a 的写入。 这个版本可以（错误地）打印一个空字符串而不是“hello, world”。

​	另一个不正确的习惯用法是忙于等待一个值，如：

```go
var a string
var done bool

func setup() {
	a = "hello, world"
	done = true
}

func main() {
	go setup()
	for !done {
	}
	print(a)
}
```

​	同上，不能保证，在 main 中，观察到对done的写入意味着观察到对 a 的写入，所以这个程序也可能打印出一个空字符串。

​	更糟糕的是，无法保证 main 会观察到对 done 的写入，因为两个线程之间没有同步事件。main 中的循环不能保证完成。

​	这个主题有更微妙的变体，比如这个程序。

```go
type T struct {
	msg string
}

var g *T

func setup() {
	t := new(T)
	t.msg = "hello, world"
	g = t
}

func main() {
	go setup()
	for g == nil {
	}
	print(g.msg)
}
```

​	即使 main 观察到 g != nil 并退出它的循环，也不能保证它会观察到 g.msg 的初始化值。

​	在所有这些示例中，解决方案都是相同的：使用显式同步。