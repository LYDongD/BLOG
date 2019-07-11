## go 线程

1 goroutine

```
func say(s string) {
    for i := 0; i < 5; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Println(s)
    }
 }

//go func 创建新线程并在新线程执行函数
func main(){
    go say("world")
    say("hello")
}

```

2 信道

信道实现go线程之间的通信
* ch <- v
* v := <- ch

```
 //创建int类型信道 c := make(chan int)
 //带缓冲信道: c := make(chan int, 2) 

 func sum(nums []int, c chan int) {
     sum := 0
     for _, num := range nums {
        sum += num
     }
     c <- sum
 }


//channel是天然阻塞的，相同的逻辑，java需要通过CountDownLatch等待所有线程执行完毕
func main(){
    nums := []int{7, 2, 8, -9, 4, 0}
    c := make(chan int)

    go sum(nums[:len(nums)/2], c)
    go sum(nums[len(nums)/2:], c)

    x ,y := <- c, <- c 
    fmt.Println(x, y, x + y)
}

//信道关闭和迭代

func fibonacco(n int, c chan int) {
    x, y := 0, 1
    for i := 0; i < n; i++ {
        c <- x
        x, y = y, x + y
    }

    close(c)
}

func main(){

    c := make(chan int, 10)
    go fibonacco(cap(c), c)
    for i := range c {
        fmt.Println(i)
    }
}

//信道操作，两个goroutine读写
//select可以监听多个信道操作，随机选择一个ready的信道
func fibinnaci(c, quit chan int) {
    x, y := 0, 1
    for {
        select {
        case c <- x:
            x, y = y, x + y
        case <- quit :
            fmt.Println("quit")
            return
        }
    }
}

func main(){

    c := make(chan int)
    quit := make(chan int)

    go func() {
        for i := 0; i < 10; i++ {
            fmt.Println(<- c)
        }

        quit <- 0
    }()

    fibinnaci(c, quit)

}


//select的默认行为, 执行dfault

func main(){
    tick := time.Tick(100 * time.Millisecond)
    boom := time.After(500 * time.Millisecond)

    for {
        select {
        case <- tick:
            fmt.Println("tick.")
        case <- boom:
            fmt.Println("boom!")
            return 
        default:
            fmt.Println("    .")
            time.Sleep(50 * time.Millisecond)
        }
    }
}

```

3 实例

* 等价二叉查找树

```
//定义二叉树节点
type TreeNode struct {
    Val int
    Left *TreeNode
    Right * TreeNode
}


//递归结束后关闭chan 
func Walk(root *TreeNode, c chan int) {
    sendValue(root, c)
    close(c)
}

//preorder recurstion, 递归数据发送到chan
func sendValue(root *TreeNode, c chan int) {
    if root != nil {
        sendValue(root.Left, c)
        c <- root.Val
        sendValue(root.Right, c)
    }
}

//在两个线程内递归数，在另外一个线程内比较序列，当chan关闭时，比较结束
func Same(t1 *TreeNode, t2 *TreeNode) bool{
    c1 := make(chan int)
    c2 := make(chan int)
    go Walk(t1, c1)
    go Walk(t2, c2)
    for i := range c1 {
        if i != <- c2 {
            return false
        }
    }

    return true
}



```

4 超时

```
func main() {
	c := make(chan int)
	o := make(chan bool)
	go func() {
		for {
			select {
				case v := <- c:
					println(v)
				case <- time.After(5 * time.Second):
					println("timeout")
					o <- true
					break
			}
		}
	}()
	<- o
}


```
