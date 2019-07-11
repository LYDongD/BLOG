## 错误

1 错误

* 错误error是golang的内建接口
* 自定义错误可通过实现Error方法来自定义错误内容输出

```
type MyError struct {
    when time.Time
    what string
}

func (e *MyError) Error() string {
    return fmt.Sprintf("at %v, %s", e.when, e.what)
}

func run() error {
    return &MyError{
        time.Now(),
        "error occurs",
    }
}


func main(){
    if err := run(); err != nil {
        fmt.Println(err)
    }
}

```

## 异常处理

```

import (
    "fmt"
    "os"
)

var user = os.Getenv("USER")

//throw panic
func makePanic() {

    if user != "liam" {
        panic("no value for user")
    }
}

//recover panic on defer
func throwsPanic(f func()) (b bool) {
    //recover if panic throws
    defer func() {
        //recover return panic msg
        x := recover()
        if x != nil {
            b = true
        }

    }()
    f()
    return
}



```
