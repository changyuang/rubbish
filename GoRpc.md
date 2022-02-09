## Rpc Package

---

只有满足以下标准的方法才可以被远程访问：

- 方法的类型是导出的
- 方法是导出的
- 方法有两个参数，并且全部是导出的
- 方法的第二个参数是一个指针
- 方法返回了error类型

```go
func (t *T) MethodName(argType T1, replyType *T2) error
```

方法的第一个参数代表调用者提供的参数； 第二个参数表示要返回给调用者的结果参数。





## 服务端流程

1、设定service的数据结构，以及对应的方法，供远程访问
2、创建对应的数据对象
3、调用ser := rpc.NewServer()创建server
4、调用ser.Register(对象)注册对象的方法
5、调用l,e := net.Listen("tcp","ip:port")创建listener
6、调用conn,err := l.Accept()接受客户端连接
7、调用go ser.ServeConn(conn)创建写成去处理请求
