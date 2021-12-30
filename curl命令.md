# curl
## 1、概述
- cURL是一个利用URL语法在命令行下工作的文件传输工具。支持文件上传和下载，是综合传输工具。按传统，习惯成cURL为下载工具，cURL还包含了用于程序开发的libcurl

## 2、常见用法
1. 查看网页源码
- 不加任何选项，默认发送get请求来获取链接内容到标准输出

	**curl www.sina.com**
- -o参数用于保存网页

	**curl -o filename www.sina.com**

2. 自动跳转
- 有的网页是自动跳转的，使用-L参数，curl会跳转到新的网址

	**curl -L www.sina.com**

3. 显示头部信息
- -i参数可以显示http response的头信息，连同网页代码一起
- -I参数则是只显示http response的头信息

	**curl -i www.sina.com**

	**curl -I www.sina.com**

4. 显示通信过程
- -v参数可以显示一次http通信的整个过程，包括端口连接和http response头信息
	
	**curl -v www.sina.com**
- --trace参数可以查看更加详细的通信过程 并写入到文件中

	**curl --trace output.txt www.sina.com**

5. 发送表单信息
- 发送表单有get和post两种方式，get方法相对简单，只要把数据附在网址后面就行

	**curl example.com/form.cgi?data=xxx**
- post方法必须把数据和网址分开，curl就要用到--data参数

	**curl -X POST --data "data=xxx" example.com/form.cgi**
- 如果你的数据没有经过表单编码，还可以让curl为你编码，参数是--data-urlencode

	**curl -X POST --data-urlencode "data=xxx" example.com/form.cgi**

6. HTTP动词
- curl默认的HTTP动词是GET，使用-X参数可以支持其他动词
	
	**curl -X POST www.example.com**

	**curl -X DELETE www.example.com**

7. 文件上传
- 假定文件上传的表单是下面这样
```
 <form method="POST" enctype='multipart/form-data' action="upload.cgi">  
 <input type=file name=upload>  
 <input type=submit name=press value="OK">  
 </form>
```
- 可以用curl这样上传文件

	**curl --form upload=@localfilename --form press-OK [URL]**

8. Referer字段
- 有时你需要在http request头信息中，提供一个referer字段，表示从哪里跳转过来的

	**curl --referer http://www.example.com http://www.example.com**

9. User Agent字段
- 这个字段用来表示客户端的设备信息，服务器有时会根据这个字段，针对不同设备，返回不同格式的网页，比如手机版和桌面版

	**curl --user-agent "[User Agent]" [URL]**

10. cookie
- 使用--cookie参数，可以让curl发送cookie

	**curl --cookie "name=xxx" www.example.com**

11. 增加头信息
- 有时需要在http request之中，自行增加一个头信息， --header参数可以起到这个作用

	**curl --header "Content-Type:application/json" www.example.com**

12. HTTP认证
- 有些网域需要HTTP认证，需要使用--user参数

	**curl --user name:password example.com**
