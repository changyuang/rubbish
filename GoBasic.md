### 定义多种类型的变量

```go
//下面是正确的
var (
a int32
b float32
c string
)
//下面是错误的
var a int32,b float32,c string
```

### 变量类型

​	int、uint、uintptr在 32 位系统上通常为 32 位宽，在 64 位系统上则为 64 位宽。当你需要一个整数值时应使用 int 类型，除非你有特殊的理由使用固定大小或无符号的整数类型。

​	uint8 == byte 
​	int32 == rune

### 类型转换

T(v) 将值v转换为类型T
与 C 不同的是，Go 在不同类型的项之间赋值时需要**显式转换**。

### for循环

```go
//最普通版
for i := 1;i < 1000;i ++{}
```

```go
// 这里等价于while循环
i := 10
for i < 1000{}   
```

```go
//死循环
for{} 
```

```go
//for range切片
//返回两个值  第一个是下标，第二个是值
for i,v := range arr{}
//可以用_来忽略，如果忽略v则可以直接省略
for i,_ := range arr{} 等价与 for i := range arr{}
```

​	注：在for循环中使用匿名函数，并且匿名函数引用了for循环得到的下标或者值时，需要拷贝一份再传入匿名函数中，而不是直接将for循环中得到的下标或者值直接传入！！！！

### switch

go中的switch语句在每一个case语句中自动添加了break语句，也就是只会选择第一个满足条件的语句

```go
switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.\n", os)
	}
/*GOOS is the running program's operating system target: one of darwin, freebsd, linux, and so on. To view possible combinations of GOOS and GOARCH, run "go tool dist list".*/
```

### 结构体成员访问

1. 可以通过"."来访问
2. 如果拥有结构体对象的指针p,可以简化访问方式，直接使用p.x而不需要(*p).x

### 结构体初始化

```
type Vertex struct{
	X,Y int
}
v1 := Vertex{} // X,Y 被初始化为0值
v2 := Vertex{1,2} // X = 1,Y = 2 按顺序对应
v3 := Vertex{Y:10} // X = 0,Y = 10
v4 := &Vertex{10,20} // 创建一个指针
```

### nil切片

​	切片的零值是nil，长度和容量均为0切没有底层数组，必须需要用make创建（对于一个nil切片用append也可以，会改变对应切片的容量）

### 映射

1. nil映射
   映射的零值为nil,nil映射既没有键，也不能添加键。make函数会返回给定类型的映射，并将其初始化备用

2. 删除元素

​				delete(map,key)

3. 检查元素是否存在

​				elem,ok := map[key] //若key在map中，则ok为true；否则ok为false，如果				key不在映射中，那么elem是该元素类型的零值

### 指针在方法和函数中的区别

1. 在函数中，若函数参数为一个指针，那么必须接受一个指针
2. 在方法中，若接收者为一个指针，那么接收者可以为值也可以为指针
3. 反之亦然



### 接口

1. 空接口不等于nil接口

```go
//空接口
var i interface{}
//nil 接口
type i interface{
	M()
}
var inter i
```

### 类型断言

```go
//该语句断言接口值 i 保存了具体类型 T，并将其底层类型为 T 的值赋予变量 //t。若 i 并未保存 T 类型的值，该语句就会触发一个恐慌。
t := i.(T)

//类型断言可以返回两个值：其底层值以及一个报告断言是否成功的布尔值。
t,ok := i.(T)
//若 i 保存了一个 T，那么 t 将会是其底层值，而 ok 为 true。
//否则，ok 将为 false 而 t 将为 T 类型的零值，程序并不会产生恐慌。
```

**类型选择**

​	类型选择与一般的 switch 语句相似，不过类型选择中的 case 为类型（而非值）， 它们针对给定接口值所存储的值的类型进行比较。

```go
switch v := i.(type) { //这里的type只有在switch语句中可以使用
case T:
    // v 的类型为 T
case S:
    // v 的类型为 S
default:
    // 没有匹配，v 与 i 的类型相同
}

注意这里的switch v := i.(type) 后面没有";v"!!!!!!
```

### 管道

在使用range关键词读取channel时，应该要注意最后要close

### select语句

select 语句使一个 Go 程可以等待多个通信操作。select会阻塞到某个分支可以继续执行为止，这时就会执行该分支。当多个分支都准备好时会**随机**选择一个执行

当select中的其他分支都没有准备好时，default分支就会执行，为了在尝试发送或者接收时不发生阻塞，可使用 default 分支

```
func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		default:
			break
		}
	}
}
```

