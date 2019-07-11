## golang 02

#### 环境搭建

* brew install go
* go env 查看相关变量
* 环境变量: 将go命令添加到环境变量PATH，并创建GOPATH
	* export GOROOT=$HOMT/XXX
	* export GOPATH=$HOME/gocode/
	* export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
* go version 查看当前版本


#### GOPATH

* GOPATH不是GOROOT，也不是go的安装目录，是go命令执行依赖的一个必须的环境变量
* GOPATH允许多个目录，当有多个目录时，用冒号分割
* 默认为$HOME/go,可自定义文件夹并更改GOPATH
* GOPATH存放go源码，可执行文件和编译后的包，即src,bin,pkg
    * src 存放源代码（比如：.go .c .h .s等）
    * pkg 编译后生成的文件（比如：.a）
    * bin 编译后生成的可执行文件（为了方便，可以把此目录加入到 $PATH 变量中，如果有多个gopath，那么使用${GOPATH//://bin:}/bin添加所有的bin目录）
        * go install 执行package会生成bin下的可执行文件(main包)和pkg下的二进制文件(可被其他应用import)

#### 目录结构和文件关系

参考("https://github.com/LYDongD/build-web-application-with-golang/blob/master/zh/01.2.md")


**1 执行程序**

* 直接执行
	* go run hello.go

* 编译-执行
	* go build hello.go -> hello二进制
		* go build -o [二进制文件名] hello.go
	* ./hello 执行二进制文件

编译后，go会把相关依赖库编译进二进制文件，因此二进制文件比源代码更大，且可以直接在任何机器上运行

**2 注意事项**

* 源代码扩展名为.go,编译器根据扩展名识别为go程序
* 入口方法是main()函数
* 编译器负责加分号，开发者可省略(一行一个分号)
* 严格区分大小写
* 导入其他包的对象需要使用import引入
* 编译器不允许声明/引入但不使用

**3 几个有用的命令 **

* gofmt -w [file] 格式化文件
* godoc -http:=8088 启动本地golang文档服务器
* godoc -src fmt Println 查看函数源码
* godoc fmt Println 查看函数文档
* godoc net/http 查看包内函数文档

其他参考("https://github.com/LYDongD/build-web-application-with-golang/blob/master/zh/01.3.md")
